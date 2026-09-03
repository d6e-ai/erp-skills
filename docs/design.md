# ERP Skills design

Status: Accepted

## Purpose

This repository provides Agent Skills for building Japan and Singapore ERP systems on d6e.

The legacy `trebit` system is retired. Do not continue its implementation or recreate a ledger on d6e that uses Git-committed YAML as its write model. Carry forward only valid accounting concepts, including double-entry bookkeeping, balanced journals, multiple currencies, trial balances, income statements, and balance sheets, and redesign them for d6e.

## Language policy

English is the default language for the repository, general documentation, shared skills, Singapore skills, migration skills, and integration skills. Japan-specific skill content under `skills/jp/` may use Japanese when it improves regulatory accuracy and usability.

## d6e-first architecture

PostgreSQL tables inside each d6e workspace are the ERP system of record. Skills combine the following d6e capabilities when generating an implementation.

| Concern | d6e capability |
| --- | --- |
| Schemas, master data, documents, ledgers, retrieval, and aggregation | `d6e_sql` / built-in `d6e-sql` skill |
| Journal generation, validation, state transitions, and atomic updates | STF |
| Multi-step approval, import, reconciliation, notification, and external integration | Workflow |
| Access control by user, duty, and STF | Policy / Policy Group |
| External accounting, invoicing, and banking services | d6e SaaS integrations / Effect |

Do not create a separate database, ORM, API server, or custom ledger engine for ordinary ERP functionality. If d6e SQL cannot represent a requirement, record the constraint and alternatives as a design decision instead of silently adding another platform.

### SQL design requirements

- Design for d6e workspace isolation and table-name rewriting.
- Use `d6e_sql` as the normal path for DDL, DML, retrieval, and aggregation. Follow the built-in `d6e-sql` skill for SQL patterns.
- Keep table names at or below 23 characters so the workspace prefix still fits within PostgreSQL's identifier limit.
- Do not assume that d6e automatically adds system columns. The current implementation requires explicit columns such as `id`, `created_at`, `updated_at`, and `deleted_at`. If the target instance behaves differently, trust observed execution results.
- Do not use floating-point numbers for money. Use integer minor units or `NUMERIC` with explicit precision.
- Make accounting events, ledger posting, period close, reversals, and corrections traceable and reproducible.
- Defend journal balance, currency, accounting period, and referenced-record existence with both database constraints and STF validation.
- Define the required Policy for select, insert, update, and delete operations. Default to denial when authorization is absent.
- Expose recurring business updates through an STF or Workflow with input validation and authorization instead of relying on arbitrary update SQL from a conversation.

## Repository structure

The initial structure has four areas: shared, Japan, Singapore, and migration.

```text
skills/
├── shared/
│   └── erp-core/
│       ├── SKILL.md
│       └── references/
│           ├── d6e-architecture.md
│           ├── accounting-model.md
│           ├── authorization.md
│           ├── statutory-parameters.md
│           └── workflow-patterns.md
│
├── jp/
│   ├── erp-jp-accounting/
│   │   ├── SKILL.md
│   │   └── references/
│   │       ├── chart-of-accounts.md
│   │       ├── consumption-tax.md
│   │       ├── invoicing.md
│   │       ├── payroll.md
│   │       ├── parameter-inventory.md
│   │       ├── statutory-reporting.md
│   │       └── record-retention.md
│   └── erp-jp-integrations/
│       ├── SKILL.md
│       └── references/
│           ├── freee.md
│           ├── moneyforward.md
│           └── banking.md
│
├── sg/
│   ├── erp-sg-accounting/
│   │   ├── SKILL.md
│   │   └── references/
│   │       ├── chart-of-accounts.md
│   │       ├── gst.md
│   │       ├── invoicing.md
│   │       ├── payroll.md
│   │       ├── parameter-inventory.md
│   │       ├── statutory-reporting.md
│   │       └── record-retention.md
│   └── erp-sg-integrations/
│       ├── SKILL.md
│       └── references/
│           ├── invoicenow.md
│           ├── banking.md
│           └── government-services.md
│
└── migration/
    └── erp-migrate-trebit/
        ├── SKILL.md
        └── references/
            ├── data-mapping.md
            └── reconciliation.md
```

This tree describes the planned structure. Do not create empty directories. Add each skill and reference only when its content is implemented.

## Skill boundaries

### `erp-core`

Country-neutral d6e implementation principles, including SQL schema design, the core double-entry model, master data, authorization, auditing, STF, and Workflow composition.

### `erp-jp-accounting` and `erp-sg-accounting`

Country-specific ERP accounting. Initially separate tax, invoicing, payroll, statutory reporting, and retention requirements into `references/`, not individual skills. Promote an area to another skill only after it demonstrates an independent trigger, implementation workflow, or large context requirement.

### `erp-jp-integrations` and `erp-sg-integrations`

Mappings between external services and the d6e SQL data model, synchronization direction, identifiers, idempotency, reconciliation, and error handling. Do not duplicate general external API references; use an available built-in d6e SaaS skill.

### `erp-migrate-trebit`

A temporary skill for moving legacy `trebit` data into the d6e SQL schema and reconciling balances and reports. Normal operation of a new ERP must not depend on it.

## Skill package shape

Use the following shape by default.

```text
erp-sg-accounting/
├── SKILL.md
├── references/
│   ├── topic-a.md
│   └── topic-b.md
└── assets/              # only when a real artifact must be copied into output
```

- Put invocation conditions, key decision steps, critical invariants, and reference routing in `SKILL.md`.
- Put detailed regulations, SQL schemas, journal examples, workflows, and acceptance criteria in `references/`.
- Record authoritative sources and review dates for country-specific rules, and add effective dates to time-dependent values.
- Collect statutory rates, recoverable proportions, monetary thresholds, deadlines, rate-table versions, and form versions in each country's `references/parameter-inventory.md`. Regulatory prose must refer to stable inventory keys instead of duplicating values.
- A repository inventory is a machine-processable seed for change detection and review. The runtime system of record is a d6e SQL statutory-parameter master. Distinguish transaction dates, tax-period start dates, payment dates, filing dates, and other applicability anchors, and select exactly one approved effective-period row.
- Treat `tax_codes` as a transaction-classification master, not another authoritative source for statutory rates. When a calculation snapshots a rate, record the referenced statutory-parameter row and version.
- Do not overwrite an existing statutory value after an amendment. Add a future row and complete regression tests and approval before its effective date.
- Create `assets/` only for real artifacts, such as an invoice template, that generated output must copy.
- Skill-local `scripts/` are not installed into d6e. Do not make them runtime dependencies.

## Naming

- Shared skill: `erp-core`
- Japan skills: `erp-jp-*`
- Singapore skills: `erp-sg-*`
- Migration skills: `erp-migrate-*`
- Use only lowercase letters, digits, and hyphens in skill names, with a maximum of 64 characters.
- Do not use `d6e-*`; that prefix is reserved for built-in d6e skills.

## Deferred decisions

Decide the following while implementing the relevant skill:

- The first country and business workflow to implement
- The standard chart of accounts and company-specific extension model
- Concrete mappings from accounting events to journals
- Synchronization direction and source of truth for each external service
- The actual legacy Trebit data formats, record counts, and reconciliation criteria
