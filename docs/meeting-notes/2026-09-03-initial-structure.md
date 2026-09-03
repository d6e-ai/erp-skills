# ERP Skills initial structure

Date: 2026-09-03

## Decisions

- `trebit` は廃止し、今後の基幹システムは d6e を基盤として構築する。
- ERP データの定義、保存、検索、集計には d6e の `d6e_sql` と組み込み `d6e-sql` スキルを使う。
- 業務ロジックは SQL に加えて STF、Workflow、Policy、既存 SaaS integration を適切に組み合わせる。
- 国別スキルは `skills/jp/` と `skills/sg/` に分ける。
- 各国の初期スキルは `accounting` と `integrations` に相当する大項目に留める。
- 税、請求、給与、法定報告、記録保存などの詳細は、まず各スキルの `references/` に分ける。
- 個別領域は、独立した利用場面や大きなコンテキストが確認できた場合だけ別スキルにする。
- Trebitからのデータ移行は `erp-migrate-trebit` に隔離し、新規ERPの通常運用を依存させない。

現在の規範的な構成と実装原則は [`docs/design.md`](../design.md) を正本とする。
