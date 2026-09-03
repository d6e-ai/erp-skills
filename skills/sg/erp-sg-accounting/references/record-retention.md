# Singapore accounting record retention

Official information reviewed: 2026-09-03. Re-check the rules for the entity and document type before applying deletion policies.

ACRAとIRASの保存年数は[parameter-inventory.md](parameter-inventory.md)を参照する。起算点や対象が異なるため、同じ年数でも一つの削除ルールへまとめない。

Official sources:

- [ACRA: Company directors’ duties and key obligations](https://www.acra.gov.sg/manage/companies/legal-requirements-common-offences/directors-duties/)
- [IRAS: Keeping records](https://www.iras.gov.sg/taxes/goods-services-tax-%28gst%29/basics-of-gst/invoicing-price-display-and-record-keeping/keeping-records)

## SQL and storage design

証憑とretention policyに次を持たせる。

- entity、document type、transaction date、financial year、GST accounting period
- supplier/customer、amount、GST classification
- d6e storage file ID、original filename、MIME type、digest
- source、received/issued timestamp、related invoice/payment/journal IDs
- retention rule ID、retention start、retain until、legal hold
- replacement、credit/debit note、original documentとの関係

一律「作成日から5年」と計算しない。ACRAとIRASで起算点や対象が異なるため、rule IDごとに起算イベントを持つ。

## Controls

- `deleted_at`だけで法定保存を満たすと仮定しない。
- retain-until以前の物理削除をPolicyと削除STFで拒否する。
- GST return、financial statements、XBRL filingで使用したsnapshotとsource recordsの関係を保持する。
- external providerへ保存を委ねる場合でも、取得可能性、完全性、移行・解約時exportを検証する。
