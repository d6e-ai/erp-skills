---
name: erp-core
description: >-
  Design and implement country-neutral ERP accounting data, controls, and workflows on d6e using d6e_sql, STF, Workflow, and Policy. Use for core ledgers, master data, posting, close, reconciliation, or d6e ERP architecture. Use erp-jp-accounting or erp-sg-accounting for country-specific rules.
---

# d6e ERP core

Design and build a reusable core ERP accounting system with PostgreSQL inside a d6e workspace as the system of record.

## Platform boundary

- Use `d6e_sql` for data definition, storage, retrieval, and aggregation. Read the built-in `d6e-sql` skill before writing SQL.
- Implement reusable business updates as saved STFs. Read the built-in `d6e-stf` skill first and validate code with `d6e_instant_run_stf`.
- Create a Workflow only when connecting multiple STFs, inputs, files, or Effects, and follow the built-in `d6e-workflow` skill.
- Design Policy and Policy Groups before DML and follow the built-in `d6e-policy` skill.
- Do not create a database, ORM, API server, or Git/YAML ledger outside d6e. If one becomes necessary, report it as a platform gap instead of implementing it implicitly.

Read [references/d6e-architecture.md](references/d6e-architecture.md) when working with current d6e SQL constraints.

## Accounting model

Define at least the following concepts:

- Legal entity or ledger unit
- Functional and transaction currencies
- Accounting periods and close states
- Accounts and normal balances
- Journal headers with multiple journal lines
- Partners, tax classifications, and source-document references
- Idempotent posting, reversal, correction, and external import

Use [references/accounting-model.md](references/accounting-model.md) for concrete table boundaries and SQL design.

Double-entry bookkeeping records a transaction across multiple accounts and requires total debits to equal total credits. For example, a credit sale with a net amount of 100 and tax of 10 records accounts receivable of 110 on the debit side, and revenue of 100 plus tax payable of 10 on the credit side. Both sides total 110.

## Non-negotiable invariants

- Never overwrite or physically delete a posted journal. Correct an error with a reversing journal and a new correct journal, preserving cross-references.
- Balance debits and credits in each journal currency. If functional-currency amounts are stored, balance those totals too.
- Reject posting into a closed period. Reopening must be a separate privileged operation.
- Prevent duplicate posting of the same external event with a unique source-system and source-key pair.
- Never use floating-point numbers for monetary calculations. Use SQL `NUMERIC`; pass strings or integer minor units across the STF boundary.
- Give every time-dependent tax rate, payroll rate, form version, and similar parameter effective start and end dates.
- Centralize statutory numbers, deadlines, and versions in country parameter inventories instead of duplicating them in STF code or regulatory prose. At runtime, read an approved, effective-dated d6e SQL master.
- Preserve the poster, posting time, source document, reversed journal, and external identifiers for auditability.

## Implementation workflow

1. Inspect the current workspace, tables, Policies, STFs, and Workflows, then distinguish reuse, migration, and new construction.
2. Confirm the legal entity, functional currency, fiscal year, required country skill, and external systems of record. Do not guess unresolved legal choices.
3. Design short table names, keys, foreign keys, `CHECK` constraints, and `UNIQUE` constraints. Execute each DDL statement separately with `d6e_sql`.
4. Create least-privilege Policy Groups and Policies per table and operation. Give each STF access only to the tables it needs.
5. Implement posting, reversal, close, reopen, and import as STFs. Keep related validation and writes in one STF transaction.
6. Create a Workflow only for multi-step work or external integration.
7. Reconcile balances, trial balance, profit and loss, financial position, tax summaries, and open items with `d6e_sql`.
8. Report observed tool results separately from unverified regulations or external connections.

Use [references/authorization.md](references/authorization.md) for authorization design and [references/workflow-patterns.md](references/workflow-patterns.md) for the STF and Workflow boundary.
Use [references/statutory-parameters.md](references/statutory-parameters.md) for the statutory-parameter data contract and amendment workflow.

## Country routing

- Use `erp-jp-accounting` for Japan tax, invoicing, payroll, retention, and statutory reporting. Its content may be in Japanese.
- Use `erp-sg-accounting` for Singapore GST, invoicing, payroll, retention, and ACRA or IRAS reporting.
- Regulatory information is time-dependent. Before an external change is written, verify the official sources identified by the relevant country skill.
