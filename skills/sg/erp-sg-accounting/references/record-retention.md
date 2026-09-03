# Singapore accounting record retention

Official information reviewed: 2026-09-03. Re-check the rules for the entity and document type before applying deletion policies.

Refer to [parameter-inventory.md](parameter-inventory.md) for ACRA and IRAS retention periods. Their scope and start points differ, so do not combine them into one deletion rule even when the number of years is the same.

Official sources:

- [ACRA: Company directors' duties and key obligations](https://www.acra.gov.sg/manage/companies/legal-requirements-common-offences/directors-duties/)
- [IRAS: Keeping records](https://www.iras.gov.sg/taxes/goods-services-tax-%28gst%29/basics-of-gst/invoicing-price-display-and-record-keeping/keeping-records)

## SQL and storage design

Store the following for evidence and retention policies:

- entity, document type, transaction date, financial year, and GST accounting period
- supplier or customer, amount, and GST classification
- d6e storage file ID, original filename, MIME type, and digest
- source, received or issued timestamp, and related invoice, payment, and journal IDs
- retention rule ID, retention start, retain-until date, and legal hold
- relationships to replacements, credit or debit notes, and original documents

Do not apply a blanket calculation of "five years from creation." ACRA and IRAS rules can have different scopes and start points, so store the anchor event for each rule ID.

## Controls

- Do not assume that `deleted_at` alone satisfies statutory retention.
- Use Policy and a deletion STF to reject physical deletion before the retain-until date.
- Preserve relationships between source records and snapshots used for GST returns, financial statements, and XBRL filings.
- When records are retained by an external provider, verify retrievability, completeness, and export on migration or termination.
