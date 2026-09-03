# Singapore invoicing and InvoiceNow

Official information reviewed: 2026-09-03. InvoiceNow implementation phases can change and must be re-checked.

## Tax invoices

発行期限、simplified tax invoiceの上限、InvoiceNowの段階日程は[parameter-inventory.md](parameter-inventory.md)を参照する。GST-registered customerへのstandard-rated supplyに発行するtax invoiceには少なくとも次を保持する。

- “Tax Invoice”の表示
- supplier name/addressとGST registration number
- invoice dateと一意なinvoice number
- customer name/address
- goods/services description
- GST rate
- total excluding GST、total GST、total including GST
- exempt、zero-rated、その他supplyの区分別gross amount

SGD以外のinvoiceでは、少なくとも税抜合計、税込合計、GST payableをSGDへ換算し、承認されたrate sourceを追跡する。

Official source:

- [IRAS: Invoicing customers](https://www.iras.gov.sg/taxes/goods-services-tax-%28gst%29/basics-of-gst/invoicing-price-display-and-record-keeping/invoicing-customers)

## InvoiceNow

InvoiceNowはPeppol標準を基盤とするSingaporeのe-invoicing networkで、GST invoice dataのIRAS送信要件は段階導入される。対象日をコードへ固定せず、entityごとにphase、obligation start、exemption、Peppol ID、providerを保持する。

Official sources:

- [IRAS: GST InvoiceNow Requirement](https://www.iras.gov.sg/taxes/goods-services-tax-%28gst%29/gst-invoicenow-requirement)
- [IMDA: InvoiceNow](https://www.imda.gov.sg/invoicenow)

## d6e workflow

1. SQL上でinvoiceとtax totalsを確定する。
2. STFで必須項目、GST classification、SGD換算、roundingを検証する。
3. InvoiceNow payloadを、元帳とは別のoutbound recordとして作る。
4. Effectまたは外部providerで送信し、provider message IDとresponseを保存する。
5. 再送は同じidempotency keyを使い、仕訳を再生成しない。
6. credit/debit noteはoriginal invoiceとoriginal journalへ関連付ける。

送信成功を会計転記成功の代用にせず、両方の状態を独立して追跡する。
