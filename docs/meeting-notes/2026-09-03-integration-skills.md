# Integration skills and Trebit boundary

Date: 2026-09-03

## Decision

- Implement `erp-jp-integrations` and `erp-sg-integrations` as usable skills rather than empty planned directories.
- Do not create or retain an `erp-migrate-trebit` skill. The new ERP inherits valid accounting concepts through `erp-core`, not Trebit-specific migration or runtime behavior.
- Keep provider API details in matching built-in d6e SaaS skills. ERP integration skills own object mappings, source-of-truth decisions, synchronization state, idempotency, correction handling, and reconciliation.
- Store integration control data in d6e SQL and keep credentials in the approved d6e or provider credential store.
- Require scoped uniqueness and atomic claim or lease transitions for inbox, outbox, and source-version items. Do not automatically retry an indeterminate external write unless the endpoint guarantees idempotency.
- Keep external submission or synchronization status independent from d6e accounting-posting status.

This decision supersedes the Trebit migration-skill portion of [the initial structure decision](2026-09-03-initial-structure.md). [`docs/design.md`](../design.md) is the normative source for the accepted architecture.
