# ERP Skills design

Status: Accepted

## Purpose

このリポジトリは、d6e を基盤として日本およびシンガポール向けの基幹システムを構築するための Agent Skills を提供する。

旧 `trebit` は廃止する。`trebit` の実装を継続開発したり、Git コミットされた YAML を write model とする台帳を d6e 上に再現したりしない。複式簿記、仕訳の均衡、多通貨、試算表、損益計算書、貸借対照表など、有効な会計概念だけを d6e 向けの設計へ引き継ぐ。

## d6e-first architecture

ERP データの system of record は d6e ワークスペース内の PostgreSQL テーブルとする。スキルは次の d6e 機能を組み合わせて実装案を生成する。

| Concern | d6e capability |
| --- | --- |
| スキーマ、マスタ、伝票、元帳、検索、集計 | `d6e_sql` / built-in `d6e-sql` skill |
| 仕訳生成、検証、状態遷移、原子的な更新 | STF |
| 承認、取込、照合、通知、外部連携の複数ステップ処理 | Workflow |
| ユーザー、職務、STFごとのアクセス制御 | Policy / Policy Group |
| 会計・請求・銀行などの外部サービス | d6e SaaS integrations / Effect |

通常の ERP 機能のために、d6e と別のデータベース、ORM、API サーバー、または独自台帳エンジンを設けない。d6e の SQL で表現できない要件が見つかった場合は、別基盤を暗黙に追加せず、制約と代替案を設計判断として記録する。

### SQL design requirements

- d6e のワークスペース分離とテーブル名変換を前提とする。
- DDL、DML、検索、集計は `d6e_sql` を基本経路とし、SQLパターンはd6e組み込みの `d6e-sql` スキルに従う。
- テーブル名は、ワークスペース接頭辞を含めて PostgreSQL の識別子長制限に収まるよう、23文字以内を原則とする。
- d6e がシステム列を自動付与すると仮定しない。現行実装では必要な `id`、`created_at`、`updated_at`、`deleted_at` を明示的に定義する。対象インスタンスで挙動が変わっている場合は、実行結果を優先する。
- 金額には浮動小数点数を使わず、通貨の最小単位または精度を明示した `NUMERIC` を使う。
- 会計イベント、元帳への転記、期間締め、取消・訂正は、追跡可能で再検証できるデータモデルにする。
- 仕訳の借方・貸方合計、通貨、会計期間、参照先の存在などをデータベース制約と STF の両方で防御する。
- 読み取り、登録、更新、削除ごとに必要な Policy を定義し、無許可時は既定拒否とする。
- 継続利用する業務更新は、対話から任意の更新SQLを直接実行させるのではなく、入力検証と認可を備えたSTFまたはWorkflowとして公開する。

## Repository structure

初期構成は、共通、日本、シンガポール、移行の4領域とする。

```text
skills/
├── shared/
│   └── erp-core/
│       ├── SKILL.md
│       └── references/
│           ├── d6e-architecture.md
│           ├── accounting-model.md
│           ├── authorization.md
│           ├── statutory-parameters.md
│           └── workflow-patterns.md
│
├── jp/
│   ├── erp-jp-accounting/
│   │   ├── SKILL.md
│   │   └── references/
│   │       ├── chart-of-accounts.md
│   │       ├── consumption-tax.md
│   │       ├── invoicing.md
│   │       ├── payroll.md
│   │       ├── parameter-inventory.md
│   │       ├── statutory-reporting.md
│   │       └── record-retention.md
│   └── erp-jp-integrations/
│       ├── SKILL.md
│       └── references/
│           ├── freee.md
│           ├── moneyforward.md
│           └── banking.md
│
├── sg/
│   ├── erp-sg-accounting/
│   │   ├── SKILL.md
│   │   └── references/
│   │       ├── chart-of-accounts.md
│   │       ├── gst.md
│   │       ├── invoicing.md
│   │       ├── payroll.md
│   │       ├── parameter-inventory.md
│   │       ├── statutory-reporting.md
│   │       └── record-retention.md
│   └── erp-sg-integrations/
│       ├── SKILL.md
│       └── references/
│           ├── invoicenow.md
│           ├── banking.md
│           └── government-services.md
│
└── migration/
    └── erp-migrate-trebit/
        ├── SKILL.md
        └── references/
            ├── data-mapping.md
            └── reconciliation.md
```

このツリーは計画上の構成であり、空ディレクトリは作らない。各スキルと参照資料は、内容を作成する段階で追加する。

## Skill boundaries

### `erp-core`

国に依存しない d6e 実装原則を扱う。SQL スキーマ設計、複式簿記の基本モデル、マスタ、権限、監査、STF、Workflow の構成を含む。

### `erp-jp-accounting` and `erp-sg-accounting`

国別の基幹会計を扱う。税、請求、給与、法定報告、保存要件は、当初は個別スキルにせず `references/` へ分離する。個別領域が独立したトリガー、実装フロー、または大きなコンテキストを必要とすることが確認できた場合のみ、別スキルへ昇格する。

### `erp-jp-integrations` and `erp-sg-integrations`

外部サービスと d6e SQL データモデルの対応、同期方向、識別子、冪等性、照合、エラー処理を扱う。外部APIの一般的なリファレンスは複製せず、利用可能な d6e 組み込みSaaSスキルを参照する。

### `erp-migrate-trebit`

旧 `trebit` のデータを d6e SQL スキーマへ移行し、残高と帳票を照合する期間限定のスキルとする。新規ERPの通常運用はこのスキルへ依存させない。

## Skill package shape

各スキルは、原則として次の形を取る。

```text
erp-sg-accounting/
├── SKILL.md
├── references/
│   ├── topic-a.md
│   └── topic-b.md
└── assets/              # 出力へコピーする実物が必要な場合のみ
```

- `SKILL.md` には適用条件、主要な判断手順、重要な不変条件、必要な参照ファイルへの案内を書く。
- 詳細な制度、SQLスキーマ、仕訳例、ワークフロー、受入条件は `references/` に置く。
- 国別ルールには根拠となる公式情報と確認日を記録し、時点依存の値には適用開始日を付ける。
- 税率、控除割合、金額上限、期限、率表・様式バージョンなどは、国別の `references/parameter-inventory.md` に集約する。制度説明の参照ファイルは値を重複せず、世代をまたいで安定したインベントリのkeyを参照する。
- リポジトリ上のインベントリは変更把握とレビューのための機械処理可能なseedであり、実行時の正本はd6e SQLの法定パラメーターマスタとする。取引日、課税期間開始日、支払日、申告日などの適用基準を区別し、承認済みの有効期間行を一意に選ぶ。
- `tax_codes`は取引分類masterとし、法定rateの正本を重複保持しない。計算結果へrateをsnapshotするときは、参照した法定パラメーター行とversionを記録する。
- 法改正では既存値を上書きせず、将来行を追加し、適用開始前に回帰テストと承認を完了する。
- `assets/` は請求書雛形など、生成結果へ実際にコピーするものがある場合だけ作る。
- スキル内の `scripts/` は d6e へのインストール時に利用されないため、実行時の必須要素にしない。

## Naming

- 共通スキル: `erp-core`
- 日本向け: `erp-jp-*`
- シンガポール向け: `erp-sg-*`
- 移行用: `erp-migrate-*`
- スキル名は小文字、数字、ハイフンのみを使い、64文字以内とする。
- `d6e-*` は d6e 組み込みスキル用の予約接頭辞なので使用しない。

## Deferred decisions

次の内容は、個別スキルを実装するときに決める。

- 最初に実装する国と業務フロー
- 標準勘定科目と企業ごとの拡張方法
- 会計イベントから仕訳への具体的なマッピング
- 外部サービスごとの同期方向と source of truth
- Trebitから移行する実データの形式、件数、照合基準
