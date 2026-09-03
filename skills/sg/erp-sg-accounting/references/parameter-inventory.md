# Singapore statutory parameter inventory

Checked on: 2026-09-03. This is a review and d6e SQL seed inventory, not the runtime system of record. Every repository row remains `pending` until independently approved for the target d6e workspace.

Status is as of 2026-09-03. Entity classification, employee status, transaction type, and transitional rules remain separate conditions. A blank effective date is unverified and is allowed only for a `verify` row.

## Parameter values

`effective_from` is inclusive and `effective_to` is exclusive. Evaluate the interval against the date identified by `anchor_type`.

| Key | Version | Type | Value | Unit | Scope key | Anchor type | Effective from | Effective to |
| --- | --- | --- | ---: | --- | --- | --- | --- | --- |
| `sg.gst.standard` | `2024.1` | numeric | 0.09 | ratio | `standard_supply` | `time_of_supply` | 2024-01-01 | |
| `sg.invoice.issue_days` | `unverified` | numeric | 30 | days | `applicable_tax_invoice` | `time_of_supply` | | |
| `sg.invoice.simple_max` | `unverified` | numeric | 1000 | SGD incl. GST | `simplified_tax_invoice` | `invoice_date` | | |
| `sg.records.gst.minimum` | `unverified` | numeric | 5 | years | `gst_records` | `record_period_end` | | |
| `sg.records.acra.minimum` | `unverified` | numeric | 5 | years | `accounting_records` | `financial_year_end` | | |
| `sg.cpf.ow_ceiling` | `2026.1` | numeric | 8000 | SGD/month | `covered_employee` | `pay_date` | 2026-01-01 | |
| `sg.cpf.annual_salary_ceiling` | `2026.1` | numeric | 102000 | SGD/year | `covered_employee` | `pay_date` | 2026-01-01 | |
| `sg.invoicenow.phase_rules` | `2026-08-24` | dataset | `sg-invoicenow-rules-2026-08-24` | dataset | `gst_registered_business` | `rule_specific` | 2025-11-01 | |
| `sg.cpf.contribution_rates` | `2026.1` | dataset | `sg-cpf-rates-2026` | dataset | `citizen_and_spr` | `pay_date` | 2026-01-01 | 2027-01-01 |
| `sg.cpf.contribution_rates` | `2027.1` | dataset | `sg-cpf-rates-2027` | dataset | `citizen_and_spr` | `pay_date` | 2027-01-01 | |
| `sg.acra.xbrl.taxonomy` | `2026.1` | version | `2026-v1.0` | version | `xbrl_filing` | `filing_date` | 2026-02-25 | |

## Governance metadata

| Key | Version | Status | Approval | Checked on | Supersedes | Consumers | Scope | Official source |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `sg.gst.standard` | `2024.1` | active | pending | 2026-09-03 | | `sg_gst_calc` | Standard-rated supplies; zero-rated and exempt rules are separate | [IRAS GST rate][iras-gst-rate] |
| `sg.invoice.issue_days` | `unverified` | verify | pending | 2026-09-03 | | `sg_invoice_validation` | General deadline from time of supply for applicable tax invoices; confirm legal effective date | [IRAS invoicing][iras-invoicing] |
| `sg.invoice.simple_max` | `unverified` | verify | pending | 2026-09-03 | | `sg_invoice_validation` | Maximum total payable for a simplified tax invoice; confirm legal effective date | [IRAS invoicing][iras-invoicing] |
| `sg.records.gst.minimum` | `unverified` | verify | pending | 2026-09-03 | | `sg_retention_policy` | GST invoices and supporting records; confirm the retention anchor and legal effective date | [IRAS invoicing][iras-invoicing] |
| `sg.records.acra.minimum` | `unverified` | verify | pending | 2026-09-03 | | `sg_retention_policy` | Accounting records; confirm the financial-year anchor and legal effective date | [ACRA directors' duties][acra-duties] |
| `sg.cpf.ow_ceiling` | `2026.1` | active | pending | 2026-09-03 | | `sg_cpf_calc` | Maximum monthly Ordinary Wages attracting CPF contributions | [CPF OW ceiling][cpf-ow] |
| `sg.cpf.annual_salary_ceiling` | `2026.1` | active | pending | 2026-09-03 | | `sg_cpf_calc` | Annual salary ceiling; Additional Wage ceiling is this amount less total OW subject to CPF | [CPF wage rules][cpf-wages] |
| `sg.invoicenow.phase_rules` | `2026-08-24` | active | pending | 2026-09-03 | | `sg_invoicenow_eligibility` | Versioned rule dataset; use the typed rule rows below, not prose stored in a scalar value | [IRAS InvoiceNow][iras-invoicenow] |
| `sg.cpf.contribution_rates` | `2026.1` | active | pending | 2026-09-03 | | `sg_cpf_calc` | Complete employer and employee matrix by age, wage, citizenship and SPR year | [CPF current rates][cpf-rates] |
| `sg.cpf.contribution_rates` | `2027.1` | future | pending | 2026-09-03 | `sg.cpf.contribution_rates@2026.1` | `sg_cpf_calc` | Published changes for employees above 55 to 65; import the complete official matrix | [CPF 2027 changes][cpf-2027] |
| `sg.acra.xbrl.taxonomy` | `2026.1` | active | pending | 2026-09-03 | | `sg_xbrl_generation` | Preserve taxonomy and validation/business-rule versions with generated output | [ACRA requirements][acra-xbrl] |

## InvoiceNow dataset seed

The dataset key above identifies this rule set. Store implementation dates separately from typed predicates so STFs can compare dates and amounts without parsing prose.

| Rule ID | Implementation date | Cohort | Status |
| --- | --- | --- | --- |
| `new_voluntary_6m` | 2025-11-01 | Newly incorporated company applying for voluntary GST registration | active |
| `new_voluntary_all` | 2026-04-01 | Business applying for voluntary GST registration | active |
| `new_compulsory_all` | 2028-04-01 | Business applying for compulsory GST registration | future |
| `existing_200k` | 2028-04-01 | Existing GST-registered business | future |
| `existing_1m` | 2029-04-01 | Existing GST-registered business | future |
| `existing_4m` | 2030-04-01 | Existing GST-registered business | future |
| `existing_over_4m` | 2031-04-01 | Existing GST-registered business | future |

| Rule ID | Predicate | Operator | Type | Value | Unit or basis |
| --- | --- | --- | --- | ---: | --- |
| `new_voluntary_6m` | `registration_method` | eq | enum | `voluntary` | |
| `new_voluntary_6m` | `incorporation_age` | lte | numeric | 6 | months at application |
| `new_voluntary_all` | `registration_method` | eq | enum | `voluntary` | |
| `new_compulsory_all` | `registration_method` | eq | enum | `compulsory` | |
| `existing_200k` | `annual_supplies_2025` | lte | numeric | 200000 | SGD; prescribed accounting periods ending in 2025 |
| `existing_1m` | `annual_supplies_2025` | lte | numeric | 1000000 | SGD; prescribed accounting periods ending in 2025 |
| `existing_4m` | `annual_supplies_2025` | lte | numeric | 4000000 | SGD; prescribed accounting periods ending in 2025 |
| `existing_over_4m` | `annual_supplies_2025` | gt | numeric | 4000000 | SGD; prescribed accounting periods ending in 2025 |

The implementation date and the entity's GST registration effective date are distinct. Preserve the selected rule ID, dataset version, measured supplies, measurement basis, and final implementation date on the entity record.
For an existing business that matches more than one cumulative threshold, choose the earliest matching implementation date. Do not apply a later, broader threshold over an already assigned earlier phase.

## Dataset rules

- A dataset entry does not mean its detail rows have been imported. Load the complete official table, classifications, rounding rules and intervals into d6e SQL, reconcile counts and known examples, then set `approval=approved`.
- CPF contribution rates include both employer and employee shares. Do not use an `employee_rates` key for the combined dataset.
- Compute the Additional Wage ceiling from the official formula; do not treat 102000 as the employee's AW ceiling without subtracting total Ordinary Wages subject to CPF.
- Keep ACRA taxonomy, validation rules and preparation-tool versions as related but distinct parameters.

## Update rules

- Keep the key stable and add a new version with a new half-open interval when a statutory value changes.
- Annual-supplies thresholds are typed classification inputs, not standalone eligibility decisions. Preserve the measurement period and exclusions with the assigned phase.
- Do not promote a `verify` row into the approved d6e runtime set until its legal effective date and complete scope are recorded.
- d6e must reject overlapping approved rows for the same key, scope and anchor.

[iras-gst-rate]: https://www.iras.gov.sg/taxes/goods-services-tax-%28gst%29/charging-gst-%28output-tax%29/when-to-charge-goods-and-services-tax-%28gst%29/prevailing-rate-of-9
[iras-invoicing]: https://www.iras.gov.sg/taxes/goods-services-tax-%28gst%29/basics-of-gst/invoicing-price-display-and-record-keeping/invoicing-customers
[acra-duties]: https://www.acra.gov.sg/manage/companies/legal-requirements-common-offences/directors-duties/
[cpf-ow]: https://www.cpf.gov.sg/service/article/what-is-the-ordinary-wage-ow-ceiling-mbr
[cpf-wages]: https://www.cpf.gov.sg/employer/employer-obligations/what-payments-attract-cpf-contributions
[iras-invoicenow]: https://www.iras.gov.sg/taxes/goods-services-tax-%28gst%29/gst-invoicenow-requirement
[cpf-rates]: https://www.cpf.gov.sg/employer/employer-obligations/how-much-cpf-contributions-to-pay
[cpf-2027]: https://www.cpf.gov.sg/service/article/what-are-the-changes-to-the-cpf-contribution-rates-for-senior-workers-that-will-take-effect-from-1-january-2027
[acra-xbrl]: https://www.acra.gov.sg/manage/companies/legal-requirements-common-offences/filing-financial-statements-in-xbrl-format/requirements-exemptions/
