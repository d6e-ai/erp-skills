# Statutory parameter inventory

Date: 2026-09-03

## Decision

- 税率、控除割合、金額上限、期限、率表・様式versionなど、法改正で変わる値を説明文やSTFへ分散させない。
- 日本とシンガポールの各accounting skillに`references/parameter-inventory.md`を置き、値、単位、適用期間、状態、一次情報、確認日を集約する。
- リポジトリ上のinventoryは変更検知・レビュー・初期投入用とし、実行時のsystem of recordはd6e SQLの適用日付きマスタとする。
- 将来施行値は`future`として先に登録できるが、承認と適用日を満たすまでSTFに選択させない。
- 法改正時は既存行を上書きせず、新しい期間行を追加して境界日前後を回帰テストする。
- keyはversionを埋め込まず世代をまたいで安定させ、versionと半開区間の適用期間を別列にする。
- 適用判定には取引日だけでなく、課税期間開始日、支払日、申告日などを識別する`anchor_type`を渡す。
- 国別inventoryは共通契約の全fieldを保持し、scalar、dataset参照、dataset明細を型付きで分離する。
- 法定rateの正本は`law_params`とし、`tax_codes`は分類と参照を保持する。

この方針の正本は[`docs/design.md`](../design.md)とする。
