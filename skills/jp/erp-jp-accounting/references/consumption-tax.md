# 消費税

Official information reviewed: 2026-09-03. Re-check before production use.

## Current baseline

現行税率と経過措置の割合は[parameter-inventory.md](parameter-inventory.md)を参照する。税率だけで課税判定を決めず、課税、軽減、ゼロ相当の扱い、非課税、不課税、免税、リバースチャージ、輸入など、取引の性質を区別する。

Official sources:

- [国税庁 No.6102 消費税の軽減税率制度](https://www.nta.go.jp/taxes/shiraberu/taxanswer/shohi/6102.htm)
- [国税庁 No.6105 課税の対象](https://www.nta.go.jp/taxes/shiraberu/taxanswer/shohi/6105.htm)
- [国税庁 No.6497 仕入税額控除のために保存する帳簿および請求書等の記載事項](https://www.nta.go.jp/taxes/shiraberu/taxanswer/shohi/6497.htm)
- [国税庁 令和8年度税制改正特集](https://www.nta.go.jp/taxes/shiraberu/zeimokubetsu/shohi/keigenzeiritsu/invoice-review/index.htm)

## SQL data requirements

`tax_codes`または同等の表に次を持たせる。

- jurisdiction=`JP`
- 業務上の税区分コードと表示名
- 売上・仕入、課税性、標準・軽減などの分類
- 合計税率、国税率、地方税率を解決する`law_params`参照
- 控除可能割合を解決する`law_params`参照と、その適用条件を示すルールID
- 税抜・税込の扱い、端数処理単位と方向
- `effective_from`、`effective_to`、根拠URL、確認日

税率や経過措置を過去行の上書きで変更しない。取引日または制度上の適用判定日を`anchor_type`とともに渡し、有効な`law_params`行を選ぶ。課税期間開始日が基準の制限を取引日だけで選ばない。

## Transaction requirements

仕訳または税明細から次を再現できるようにする。

- 課税売上・課税仕入の相手方
- 取引日、内容、税込または税抜金額
- 税率区分と軽減対象である旨
- 税額、端数処理方法
- 適格請求書等の証憑IDと登録番号履歴
- 帳簿のみ保存などの例外を使った場合の理由コード

複数税率がある取引は区分経理できる粒度で明細を分ける。

## Calculation control

請求書単位・税率単位の集計と端数処理を明示し、行ごとの税額合計と請求書合計の差を説明できるようにする。税額計算STFは、税区分マスタから取得した値、計算方式、丸め結果を返し、仕訳へ保存する。

## Do not infer

課税方式、控除可否、経過措置、課税期間、申告義務は企業条件と取引内容で変わる。不明な場合は仮の税区分で転記せず、レビュー待ち状態へ送る。
