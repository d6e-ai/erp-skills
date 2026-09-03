# Singapore statutory reporting

Official information reviewed: 2026-09-03. Company classification, exemptions, filing rules, and taxonomies change; verify them for the filing date.

## Separate reporting layers

1. Ledger and subledgers in d6e SQL
2. Entity financial statements
3. ACRA XBRL taxonomy mappings
4. IRAS GST and corporate income-tax reporting data

Do not bind an account code directly to an ACRA or IRAS field. Use a mapping table with report type, version, and effective dates.

## ACRA XBRL

ACRA distinguishes filing requirements and formats by company classification, including Full XBRL, Simplified XBRL, specialised templates, and PDF. Do not select a template before confirming the entity's classification.

Refer to [parameter-inventory.md](parameter-inventory.md) for the applicable ACRA taxonomy and start date. Do not hard-code a taxonomy version. Record the applicable version and validation rules when generating output.

Official sources:

- [ACRA: Filing financial statements in XBRL format](https://www.acra.gov.sg/manage/companies/legal-requirements-common-offences/filing-financial-statements-in-xbrl-format/)
- [ACRA: Filing requirements and exemptions](https://www.acra.gov.sg/manage/companies/legal-requirements-common-offences/filing-financial-statements-in-xbrl-format/requirements-exemptions/)

## Output traceability

- entity, UEN, financial year end, and reporting period
- ledger-close ID and extraction timestamp
- accounting framework, mapping version, and taxonomy version
- generated file ID, validation result, preparer, and approver
- drill-down key from each report line to its accounts and journals

## Validation

- Confirm that trial-balance debits and credits agree.
- Reconcile subledgers to control accounts.
- Confirm continuity of comparative figures and opening balances in the financial statements.
- Run validation rules for the applicable XBRL taxonomy version.
- Trace GST-return aggregates back to invoices, credit notes, and journals.

Do not assume d6e can submit directly to ACRA or IRAS. First generate verifiable intermediate data or files, and handle submission connectivity separately in `erp-sg-integrations`.
