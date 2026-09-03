# Repository instructions

このリポジトリで作業する前に、`docs/design.md` を読むこと。設計変更時は、合意済みの現在形を `docs/design.md` に反映し、議論や日付付きの判断記録は `docs/meeting-notes/` に分離すること。

## Platform boundary

- このリポジトリのスキルは d6e 上で基幹システムを構築するためのものである。
- データ定義、保存、検索、集計には d6e の `d6e_sql` を第一選択として利用し、SQL作成時は組み込みの `d6e-sql` スキルに従う。
- トランザクションを伴う業務ロジックには d6e STF、複数ステップの処理には Workflow、認可には Policy を利用する。
- d6e と別のデータベース、ORM、API サーバー、または台帳エンジンを、明示的な要件なしに導入しない。
- `trebit` の Git/YAML write model、CLI、REST API、ブランチ管理を新システムの前提にしない。移行に必要な知識だけを `erp-migrate-trebit` に隔離する。

## Skill organization

- `skills/shared/` は国に依存しない共通事項に使う。
- `skills/jp/` と `skills/sg/` は国別スキルを格納する。
- 各国は当初 `accounting` と `integrations` に相当する大粒度のスキルから始める。税、請求、給与、法定報告などはまず `references/` に分け、独立した発見・実行単位が必要になった場合だけ別スキルへ分割する。
- スキル名とスキルディレクトリ名は一致させ、`erp-core`、`erp-jp-*`、`erp-sg-*`、`erp-migrate-*` の命名を使う。
- d6e の組み込みスキル用に予約されている `d6e-` 接頭辞を使わない。
- `SKILL.md` は判断手順、重要な制約、参照先を簡潔に記述する。詳細な制度、データモデル、SQL パターン、受入条件は隣接する `references/` に置く。
- 参照ファイルは、それを使うスキルのディレクトリ内に置く。リポジトリ全体の共有参照へ暗黙に依存させない。
- 実体のないディレクトリ、サンプル、アセットを先回りして追加しない。

## d6e implementation constraints

- SQL テーブル名は d6e のワークスペース接頭辞を含めた PostgreSQL 識別子長制限を考慮して短くする。
- d6e がシステム列を自動付与すると仮定しない。必要な `id`、`created_at`、`updated_at`、`deleted_at` は、対象インスタンスの挙動を確認したうえで明示的に定義する。
- d6e の既定拒否型 Policy を前提に、必要な操作と主体を明示する。
- 会計データでは複式簿記の均衡、期間締め、訂正、監査証跡を SQL スキーマと STF の不変条件として扱う。
- 税率、上限額、期限、率表・様式バージョンなどの法定パラメーターを説明本文やSTFへ分散させない。国別の `references/parameter-inventory.md` に安定key、version、適用基準日、適用期間、根拠、確認日、承認状態を集約し、実行時はd6e SQLの`law_params`を正本とする。`tax_codes`は分類と参照を持ち、法定rateの別正本にしない。
- 対話から直接更新SQLを乱用せず、再利用される業務更新は検証と認可を含むSTFまたはWorkflowとして定義する。
- 外部会計サービスの API 仕様は既存の d6e 組み込み SaaS スキルを再利用し、このリポジトリでは ERP データとの対応関係と業務フローに集中する。
- d6e のスキル取り込みではスキル内の `scripts/` が実行資源として利用されないため、ランタイム処理をスクリプトに依存させない。必要な処理は SQL、STF、Workflow として記述する。
