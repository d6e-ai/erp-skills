# 日本の給与会計

Official information reviewed: 2026-09-03. Current table versions are listed in [parameter-inventory.md](parameter-inventory.md) and must be re-checked for the applicable year and employee.

給与計算そのものと、外部給与サービスで確定した結果の会計転記を分けて設計する。後者だけが要件なら、d6eで税額表を再実装しない。

Official starting points:

- [国税庁 源泉徴収義務者の方](https://www.nta.go.jp/users/gensen/)
- [国税庁 令和8年分 源泉徴収税額表](https://www.nta.go.jp/publication/pamph/gensen/zeigakuhyo2026/01.htm)
- [国税庁 令和8年分 年末調整のしかた](https://www.nta.go.jp/publication/pamph/gensen/nencho2026/01.htm)
- [日本年金機構 事業主の方](https://www.nenkin.go.jp/service/kounen/jigyosho-hiho/jigyonushi/index.html)

## SQL model

給与会計に必要なデータを、個人情報へのアクセスを最小化して保持する。

- 給与期間、支払日、従業員ID
- 基本給、手当、賞与、控除の種類別金額
- 源泉所得税、住民税、社会保険等の従業員負担と事業主負担
- 総支給、控除合計、差引支給
- 使用した税額表・率表の年度、バージョン、適用日
- 外部給与run ID、仕訳ID、支払ID

生年月日、扶養、住所、個人番号など、計算または法定手続に不要な個人情報を会計表へ複製しない。

## Posting example

総支給300,000、従業員控除60,000、差引支給240,000の場合、概念上は借方の給与費用300,000に対し、貸方の預り金60,000と未払給与または現預金240,000を計上する。事業主負担の社会保険等は別の費用と未払金として計上する。

実際の科目、認識時点、控除内訳は企業設定に従う。

## Controls

- 税額表、控除、社会保険率をコードへ固定しない。
- 給与計算runを一意にし、再取込で二重仕訳しない。
- 給与明細閲覧と総勘定元帳閲覧を別Policy Groupにする。
- 計算、承認、支払、仕訳転記の担当を追跡する。
- 訂正runは元runと元仕訳を参照し、差額または反対仕訳を説明できるようにする。
