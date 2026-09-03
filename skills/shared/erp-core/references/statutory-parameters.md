# Statutory parameter management

税率、控除割合、金額上限、提出・発行期限、給与率表、申告様式やtaxonomyのversionを、業務ロジックと制度説明から分離して管理する。

## Two layers

1. 国別`references/parameter-inventory.md`は、公式情報の変更を発見・レビューし、d6eへ投入するためのseed inventoryである。
2. d6e SQLの`law_params`または同等の短い名前の表は、STFが実行時に読むsystem of recordである。

Markdownを実行時の正本にしたり、STFがリポジトリを読めると仮定したりしない。反対に、d6e上の値だけを更新して根拠とレビュー履歴を失わない。

## Inventory contract

各エントリーには最低限次を持たせる。国別inventoryは、この契約と同じ列を持つ機械処理可能な表にする。表示上二つの表へ分ける場合も、`key`と`version`で一意に結合できなければならない。

| Field | Meaning |
| --- | --- |
| `key` | 国・制度・用途を含む安定ID。同じパラメーターの世代が変わっても変更しない |
| `version` | 同じkeyとscope内の版。`v1`などを`key`へ埋め込まない |
| `value_type` | `numeric`、`date`、`version`、`dataset`など |
| `value` | `value_type`に対応する小数、整数、日付、version、またはdataset識別子 |
| `unit` | `ratio`、`JPY`、`SGD`、`days`、`years`、`date`、`version`など |
| `scope_key` / `scope` | 機械判定用の安定scope IDと、対象取引、法人、従業員区分、例外条件の説明 |
| `anchor_type` | 適用期間を判定する日付の種類。`transaction_date`、`tax_period_start`、`pay_date`、`filing_date`など |
| `effective_from` / `effective_to` | ISO DATEの適用期間。開始を含み、終了を含まない。未確定なら空欄かつ`verify`にする |
| `status` | `active`、`future`、`expired`、`verify` |
| `approval` | `pending`、`approved`、`rejected`。repository inventoryでは独立承認前を`pending`とする |
| `source` | 政府・規制当局の一次情報URL |
| `checked_on` | 内容を人またはagentが確認した日 |
| `supersedes` | 置き換える旧エントリーの`key@version` |
| `consumers` | 使用するSTF、帳票、Workflow、検証ケース |

割合を保存するときは、`10`、`10%`、`0.10`を混在させない。d6e SQLでは`NUMERIC`の比率`0.10`など、単位ごとの表現を定める。scalarの値と適用条件を一つの自由記述へ混ぜない。複雑なCPF率表、段階適用rule、taxonomyは一つのscalarへ潰さず、安定したdataset key、dataset version、型付きの明細行を管理する。

## d6e SQL model

実行時の`law_params`では、少なくとも`jurisdiction`、`param_key`、`scope_key`、`version`、`value_type`、型付きの値、`unit`、`anchor_type`、半開区間の適用期間、`approval_status`、source URL、確認日を保持する。型付きの値は`num_value`、`date_value`、`text_value`、`dataset_key`などへ分け、`value_type`と対応する一つだけが設定されるよう`CHECK`する。同じkey、scope、anchorの期間重複は、d6eで利用可能なSQL制約または承認STF内の検査で拒否し、過去行を上書きしない。

STFは呼出側が指定した`anchor_type`と`as_of_date`に対して、承認済みの一行だけを取得する。取引日を、課税期間開始日、支払日、申告日、法人設立日などの代用にしない。該当なし、複数該当、または未承認・`verify`の行しかない場合は処理を停止する。`future`は適用日前に選ばず、`expired`は過去日付の再現時には選択可能にする。最新行ではなく、対象取引へ制度上適用される行を選ぶ。

`tax_codes`は取引分類masterであり、法定rateのsystem of recordではない。原則として承認済み`law_params`行を参照する。監査・再現のためrateを取引や税計算結果へsnapshotする場合は、参照した`law_params`のIDとversionを必ず一緒に保存し、snapshotを次回計算の正本にしない。

## Amendment workflow

1. 公式一次情報で公布・施行・経過措置を確認する。
2. 安定keyを維持したままcountry inventoryへ新versionを追加し、旧行の排他的終了日と`supersedes`を更新する。
3. 将来値は`future`としてd6e SQLへ追加し、施行前に自動選択しない。
4. 境界日前後、返品・取消、遡及処理、未確定条件の回帰ケースを実行する。
5. 独立した承認後にd6e行を`approved`とし、対象STF・帳票versionを記録する。
6. 施行後も過去取引が旧値で再現できることを確認する。

改正資料が公開されても、法令成立、施行日、企業への適用条件が未確定なら`verify`に留める。agentの判断だけで本番値を有効化しない。
