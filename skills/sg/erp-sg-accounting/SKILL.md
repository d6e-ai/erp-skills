---
name: erp-sg-accounting
description: >-
  Design and implement Singapore ERP accounting on d6e SQL, including Singapore charts of accounts, GST, tax invoices, payroll accounting, ACRA and IRAS reporting, and record retention. Use for Singapore-specific accounting requirements; use erp-sg-integrations for external service synchronization.
---

# Singapore d6e ERP accounting

Design and build accounting models, GST, invoicing, payroll journals, ACRA and IRAS reporting, and record-retention controls for Singapore entities and establishments, with d6e SQL as the system of record.

## Required platform skills

- Use `erp-core` for the shared accounting model and invariants.
- Use the built-in `d6e-sql` skill for DDL and DML, `d6e-stf` for business updates, `d6e-workflow` for multi-step operations, and `d6e-policy` for authorization.
- Do not create a standalone database outside d6e or reuse a legacy Trebit ledger.

## Confirm before implementation

Confirm the following from the existing workspace, and ask only about items that remain unknown:

- entity type, UEN, financial year end, and functional currency
- GST registration status, registration number, effective date, and GST accounting periods
- applicable InvoiceNow Requirement phase and the selected solution provider or Access Point
- whether payroll is calculated in d6e or only results from an external payroll service are posted
- whether the scope includes Singapore Citizens, Permanent Residents, and foreign employees
- whether financial statements or XBRL must be filed with ACRA, and the applicable company classification
- the source of truth for external accounting, banking, payroll, and invoicing services

Do not guess tax, employment, or filing requirements. Re-check official rules, rates, thresholds, phased implementation dates, and taxonomies at implementation time, and clearly identify entity-specific decisions that require professional review.

## References

- Chart of accounts and Singapore reporting mappings: [references/chart-of-accounts.md](references/chart-of-accounts.md)
- GST classifications, rates, and input/output tax: [references/gst.md](references/gst.md)
- Tax invoices and InvoiceNow-ready data: [references/invoicing.md](references/invoicing.md)
- CPF, payroll accounting, and itemised payslips: [references/payroll.md](references/payroll.md)
- GST rates, thresholds, deadlines, and CPF/XBRL versions: [references/parameter-inventory.md](references/parameter-inventory.md)
- ACRA XBRL and IRAS reporting: [references/statutory-reporting.md](references/statutory-reporting.md)
- Accounting and GST record retention: [references/record-retention.md](references/record-retention.md)

Read only the references relevant to the request.

## d6e implementation rules

- Store entities, periods, accounts, partners, tax codes, journals, invoices, payments, and evidence links in d6e SQL tables.
- Do not hard-code GST rates, CPF rates, wage ceilings, filing thresholds, InvoiceNow phases, or ACRA taxonomies in STFs. Use `parameter-inventory.md` as a seed for verification, then read runtime values from effective-dated d6e SQL master data with stable keys, an applicable anchor date, approval status, and source URL.
- Store UENs, GST registration numbers, Peppol IDs, and external system IDs separately by purpose. Do not conflate them into one identifier.
- For foreign-currency invoices, track the transaction-currency amount, the SGD amount for GST purposes, the rate, rate source, and rate date.
- Make persisted STFs the only update path for posting, reversal, closing, and GST aggregation, and restrict direct updates through Policy.
- Decouple InvoiceNow submission from ledger posting so a submission failure cannot create a duplicate journal entry.
- Record the reporting period, extraction time, mapping or taxonomy version, and generated file ID for every statutory output.

## Verification

Verify standard-rated, zero-rated, exempt, out-of-scope, credit-note, foreign-currency, GST-rounding, CPF-rounding, closing, and reversal cases with small numerical examples. Report executed d6e results separately from unverified external connections and decisions awaiting professional review.
