---
name: erp-core
description: >-
  Design and implement country-neutral ERP accounting data, controls, and workflows on d6e using d6e_sql, STF, Workflow, and Policy. Use for core ledgers, master data, posting, close, reconciliation, or d6e ERP architecture. Use erp-jp-accounting or erp-sg-accounting for country-specific rules.
---

# d6e ERP core

d6eのワークスペース内PostgreSQLをsystem of recordとして、再利用可能な基幹会計の中核を設計・構築する。

## Platform boundary

- データ定義、保存、検索、集計には`d6e_sql`を使う。SQLを書く前に組み込み`d6e-sql`スキルを読む。
- 再利用する業務更新は保存済みSTFにし、作成前に組み込み`d6e-stf`スキルを読み、`d6e_instant_run_stf`で検証する。
- 複数のSTF、入力、ファイル、Effectを連結するときだけWorkflowを作り、組み込み`d6e-workflow`スキルに従う。
- DMLの前にPolicyとPolicy Groupを設計し、組み込み`d6e-policy`スキルに従う。
- d6eとは別のDB、ORM、APIサーバー、またはGit/YAML台帳を作らない。必要性が生じた場合は実装せず、プラットフォームギャップとして報告する。

現在のd6e SQL制約を扱うときは[references/d6e-architecture.md](references/d6e-architecture.md)を読む。

## Accounting model

少なくとも次の概念を明示する。

- 法人または帳簿単位
- 機能通貨と取引通貨
- 会計期間と締め状態
- 勘定科目と通常残高
- 仕訳ヘッダーと複数の仕訳明細
- 取引先、税区分、原始証憑への参照
- 転記、取消、訂正、外部取込の冪等性

具体的なテーブル境界とSQL設計には[references/accounting-model.md](references/accounting-model.md)を使う。

複式簿記は、一つの取引を複数の勘定へ記録し、借方合計と貸方合計を一致させる方式である。例えば税抜100、税10の売上を掛けで計上する場合、借方の売掛金110に対し、貸方の売上100と税預り10を記録する。合計は両側とも110になる。

## Non-negotiable invariants

- 転記済み仕訳を上書きまたは物理削除しない。誤りは反対仕訳と正しい新規仕訳で訂正し、相互参照を保存する。
- 借方合計と貸方合計を仕訳通貨ごとに一致させる。基準通貨額を保持する場合は、その合計も一致させる。
- 締め済み期間への新規転記を拒否する。再オープンは独立した権限付き操作にする。
- 同じ外部イベントを二重転記しないよう、source systemとsource keyの一意性を保証する。
- 金額計算に浮動小数点数を使わない。SQLは`NUMERIC`、STFは文字列または整数最小単位を境界として扱う。
- 税率、給与率、申告様式などの時点依存値には適用開始日と適用終了日を持たせる。
- 法定の数値・期限・バージョンは国別parameter inventoryへ集約し、STFや制度説明へ重複記載しない。実行時はd6e SQLの承認済み適用日付きマスタから読む。
- 投稿者、投稿時刻、原始証憑、取消元、外部識別子を追跡できるようにする。

## Implementation workflow

1. 現在のワークスペース、既存テーブル、Policy、STF、Workflowを読み取り、再利用・移行・新設を区別する。
2. 法人、機能通貨、会計年度、必要な国別スキル、外部system of recordを確定する。不明な法的選択は推測しない。
3. 短いテーブル名、キー、外部キー、`CHECK`、`UNIQUE`を設計する。DDLは`d6e_sql`で一文ずつ実行する。
4. 最小権限のPolicy Groupと、表・操作ごとのPolicyを作る。STFにも必要な表だけを許可する。
5. 転記、取消、締め、再オープン、取込をSTFとして実装する。関連する検証と書込みを一つのSTFトランザクション内に置く。
6. 複数ステップまたは外部連携が必要な場合だけWorkflowを作る。
7. 残高、試算表、損益、財政状態、税集計、未消込を`d6e_sql`で照合する。
8. 実際に実行したツール結果と、未検証の制度・外部接続を分けて報告する。

認可設計には[references/authorization.md](references/authorization.md)、STFとWorkflowの分担には[references/workflow-patterns.md](references/workflow-patterns.md)を使う。
法定パラメーターのデータ契約と改正反映手順には[references/statutory-parameters.md](references/statutory-parameters.md)を使う。

## Country routing

- 日本の税、請求、給与、保存、法定報告は`erp-jp-accounting`を使う。
- シンガポールのGST、請求、給与、保存、ACRA/IRAS報告は`erp-sg-accounting`を使う。
- 制度情報は時点依存である。外部変更を書き込む前に、該当する国別スキルが示す公式情報を確認する。
