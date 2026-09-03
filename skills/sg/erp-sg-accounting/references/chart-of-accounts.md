# Singapore chart of accounts

Official information reviewed: 2026-09-03.

## Design principle

内部勘定科目表を、ACRA taxonomyや外部会計サービスの科目IDそのものにしない。企業のfinancial statements、GST、management reporting、ACRA XBRLへ別々にマッピングできる形でd6e SQLに保持する。

`accounts`には次を持たせる。

- entity ID、account code、English name、必要なら日本語名
- asset、liability、equity、income、expenseの分類
- debitまたはcreditのnormal side
- parent account、display order、effective dates
- financial statement line、GST control、management dimensionへのmapping key
- 外部サービスIDは別mapping table

## Recommended control accounts

- Cash and bank、trade receivables、other receivables、inventory、property/plant/equipment
- Trade payables、accruals、borrowings、output GST、input GST、CPF and payroll payables
- Share capital、reserves、retained earnings
- Revenue、cost of sales、employee costs、operating expenses、finance income/cost、tax expense

これは開始点であり、会社のaccounting frameworkや業種別表示を確定するものではない。

## Functional currency and dimensions

- entityごとにfunctional currencyを設定する。
- foreign currency amount、functional currency amount、rate、rate date、rate sourceを仕訳明細で追跡する。
- customer、supplier、employee、bank account、fixed assetはsubledgerとして保持し、勘定科目を無制限に増やさない。
- department、project、cost centreなどの分析軸は、元帳科目と分離する。

## Reporting mapping

内部科目からfinancial statementsおよびACRA XBRL taxonomyへのmappingは、taxonomy versionとeffective dates付きで保持する。taxonomy変更時に過去mappingを上書きしない。

Official starting point:

- [ACRA: Filing financial statements in XBRL format](https://www.acra.gov.sg/manage/companies/legal-requirements-common-offences/filing-financial-statements-in-xbrl-format/)
