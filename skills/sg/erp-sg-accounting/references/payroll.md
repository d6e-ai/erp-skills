# Singapore payroll accounting

Official information reviewed: 2026-09-03. CPF rate-table versions are listed in [parameter-inventory.md](parameter-inventory.md); rates, ceilings, and employment requirements must be re-checked for the applicable pay date and employee.

Separate payroll calculation from accounting entries for results finalized by an external payroll system. If the requirement is posting only, do not duplicate CPF calculations in d6e.

Official sources:

- [CPF Board: How much CPF contributions to pay](https://www.cpf.gov.sg/employer/employer-obligations/how-much-cpf-contributions-to-pay)
- [CPF contribution calculator](https://www.cpf.gov.sg/employer/tools-and-services/calculators/cpf-contribution-calculator)
- [MOM: Itemised pay slips](https://www.mom.gov.sg/employment-practices/salary/itemised-payslips)
- [MOM: Salary](https://www.mom.gov.sg/employment-practices/salary)

## Effective-dated inputs

CPF computation depends on more than one percentage. At minimum, retain or import:

- pay date and salary period
- citizenship or PR status and SPR year
- age band effective for the contribution month
- Ordinary Wages and Additional Wages
- applicable wage ceilings and contribution-table version
- employer share, employee share, and rounding results

Do not hard-code values for a specific year in an STF. Use effective-dated SQL master data that can be reconciled with the official table or calculator.

## Payroll result model

- payroll run ID, employee ID, period, and payment date
- basic salary, allowances, bonus, overtime, and other earnings
- employee CPF, other deductions, and net salary
- employer CPF and other employer costs
- rate-table version, calculation source, and approval status
- payment ID, journal ID, and reversal run ID

Separate personal information required for payslips from the aggregates required for the general ledger, and assign them to separate Policy Groups.

## Posting example

For gross salary of S$5,000, employee deductions of S$1,000, and net salary of S$4,000, the conceptual entry is a debit of S$5,000 to employee costs, a credit of S$1,000 to deductions payable, and a credit of S$4,000 to salary payable or bank. Record employer CPF and similar costs separately as a debit to employee costs and a credit to a payable.

This example explains the account structure. It does not determine CPF eligibility or rates.

## Controls

- Give each payroll run a unique identifier and reject duplicate posting.
- Store total CPF, employee share, employer share, and rounding separately.
- Track the actors responsible for calculation, approval, payment, and posting.
- Link every correction run to the original run and original journal.
