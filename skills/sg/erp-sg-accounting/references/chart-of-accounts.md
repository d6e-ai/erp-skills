# Singapore chart of accounts

Official information reviewed: 2026-09-03.

## Design principle

Do not make the internal chart of accounts identical to the ACRA taxonomy or an external accounting service's account IDs. Store it in d6e SQL so it can be mapped independently to the entity's financial statements, GST reporting, management reporting, and ACRA XBRL.

Store the following in `accounts`:

- entity ID, account code, English name, and a Japanese name when needed
- classification as asset, liability, equity, income, or expense
- normal debit or credit side
- parent account, display order, and effective dates
- mapping keys for financial-statement lines, GST controls, and management dimensions
- external service IDs in a separate mapping table

## Recommended control accounts

- cash and bank, trade receivables, other receivables, inventory, and property, plant and equipment
- trade payables, accruals, borrowings, output GST, input GST, and CPF and payroll payables
- share capital, reserves, and retained earnings
- revenue, cost of sales, employee costs, operating expenses, finance income and costs, and tax expense

This is a starting point. It does not determine the entity's accounting framework or industry-specific presentation.

## Functional currency and dimensions

- Set a functional currency for each entity.
- Track the foreign-currency amount, functional-currency amount, rate, rate date, and rate source on journal lines.
- Keep customers, suppliers, employees, bank accounts, and fixed assets as subledgers instead of expanding the chart of accounts without limit.
- Keep analytical dimensions such as department, project, and cost centre separate from ledger accounts.

## Reporting mapping

Store mappings from internal accounts to financial statements and the ACRA XBRL taxonomy with taxonomy versions and effective dates. Never overwrite historical mappings when a taxonomy changes.

Official starting point:

- [ACRA: Filing financial statements in XBRL format](https://www.acra.gov.sg/manage/companies/legal-requirements-common-offences/filing-financial-statements-in-xbrl-format/)
