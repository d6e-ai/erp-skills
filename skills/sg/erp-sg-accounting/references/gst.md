# Singapore GST

Official information reviewed: 2026-09-03. Re-check rates, thresholds, transitional rules, and scope before production use.

## Current baseline

現行GST rateと適用開始日は[parameter-inventory.md](parameter-inventory.md)を参照する。取引日だけで機械的に決めず、time of supply、transitional rule、supply classification、登録状態を考慮する。

Official sources:

- [IRAS: Prevailing rate of 9%](https://www.iras.gov.sg/taxes/goods-services-tax-%28gst%29/charging-gst-%28output-tax%29/when-to-charge-goods-and-services-tax-%28gst%29/prevailing-rate-of-9)
- [IRAS: Overview of GST rate change](https://www.iras.gov.sg/taxes/goods-services-tax-%28gst%29/gst-rate-change/gst-rate-change-for-business/overview-of-gst-rate-change)
- [IRAS: Invoicing, price display and record keeping](https://www.iras.gov.sg/taxes/goods-services-tax-%28gst%29/basics-of-gst/invoicing-price-display-and-record-keeping)

## SQL data requirements

`tax_codes`または同等の表に次を保持する。

- jurisdiction=`SG`
- standard-rated、zero-rated、exempt、out-of-scope等の分類
- output tax、input tax、reverse chargeなどのdirection
- rateとrecoverable proportionを解決する`law_params`参照、適用条件を示すrule ID
- `effective_from`、`effective_to`、根拠URL、確認日
- rounding method、currency conversion rule ID

rateやthresholdを過去行の上書きで変更しない。取引へrateをsnapshotするときは、参照した`law_params`行のIDとversionも保存する。

## Transaction requirements

GST集計から次へ遡れるようにする。

- supplier/customer、GST registration number
- supply date、invoice date、posting date、accounting period
- supply classification、exclusive amount、GST amount、inclusive amount
- transaction currencyとGST purposeのSGD amounts
- exchange rate、rate date、rate source
- tax invoice、credit/debit note、receipt、import evidence
- input tax claim statusとreview reason

## Calculation control

IRASは複数品目のGSTについて、品目ごとの計算後に合計する方法と、税抜合計へ税率を適用する方法を案内している。選択した方法とroundingを一貫して適用し、invoiceに保存する。

税額STFは有効なtax codeをSQLから選び、使用rate、exclusive amount、GST、inclusive amount、rounding deltaを返す。不明なclassificationやclaim eligibilityを自動でstandard-ratedまたはclaimableにしない。
