# ERP Skills initial structure

Date: 2026-09-03

## Decisions

- Retire `trebit` and build future ERP systems on d6e.
- Use d6e `d6e_sql` and the built-in `d6e-sql` skill for ERP data definition, storage, retrieval, and aggregation.
- Combine SQL with STF, Workflow, Policy, and existing SaaS integrations as appropriate for business logic.
- Separate country-specific skills under `skills/jp/` and `skills/sg/`.
- Begin each country with broad areas corresponding to `accounting` and `integrations`.
- Split tax, invoicing, payroll, statutory reporting, record retention, and similar details into each skill's `references/` first.
- Create another skill only after an area demonstrates an independent use case or a large context requirement.
- Isolate Trebit data migration in `erp-migrate-trebit`; do not make new ERP operation depend on it.

[`docs/design.md`](../design.md) is the normative source for the current structure and implementation principles.
