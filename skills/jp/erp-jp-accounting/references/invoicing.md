# 日本の請求・適格請求書

Official information reviewed: 2026-09-03. Re-check before production use.

制度開始日と経過措置は[parameter-inventory.md](parameter-inventory.md)を参照する。仕入税額控除に関わるため、単なるPDFレイアウトではなく、請求データ、登録履歴、税率別集計、証憑保存を一体として設計する。

Official sources:

- [国税庁 No.6498 適格請求書等保存方式](https://www.nta.go.jp/taxes/shiraberu/taxanswer/shohi/6498.htm)
- [国税庁 No.6497 帳簿および請求書等の記載事項](https://www.nta.go.jp/taxes/shiraberu/taxanswer/shohi/6497.htm)
- [国税庁 インボイス制度について](https://www.nta.go.jp/taxes/shiraberu/zeimokubetsu/shohi/keigenzeiritsu/invoice_about.htm)

## Data to retain

適格請求書では少なくとも次を構造化して保持する。

- 発行者の氏名または名称と登録番号
- 取引年月日または対象期間
- 資産または役務の内容と軽減対象の識別
- 税率ごとの税抜または税込対価合計と適用税率
- 税率ごとの消費税額等
- 書類を受ける事業者の氏名または名称
- 一意な請求番号、原本ファイル、訂正元・訂正版の関係

請求書名称だけで適格性を判定しない。必要事項が複数文書に分かれる運用では、それらの関係を追跡可能にする。

## d6e workflow

1. 請求データをSQLへ登録する。
2. 税率別集計と端数処理をSTFで確定する。
3. 発行時点で登録番号履歴と必須項目を検証する。
4. PDF等を生成する場合は生成ファイルIDを請求レコードへ結ぶ。
5. 送信、再送、取消、返還、訂正のイベントを履歴化する。
6. 売上仕訳は発行または収益認識ルールに従い、請求書生成と同じ処理で無条件に転記しない。

登録番号や税区分に問題がある仕入請求は、控除可能として自動転記せずレビュー対象にする。
