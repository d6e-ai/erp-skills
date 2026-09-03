---
name: erp-sg-integrations
description: >-
  Integrate a Singapore ERP whose system of record is d6e SQL with InvoiceNow access points, banks, ACRA, IRAS, and other external services. Use for Singapore-specific synchronization, submission, mappings, idempotency, and reconciliation; use erp-sg-accounting for accounting and statutory rules.
---

# Singapore ERP integrations

Connect a Singapore ERP to external services while keeping d6e SQL authoritative for internal accounting state.

## Required skills and boundary

- Use `erp-core` for the ledger, posting, authorization, and audit model, and `erp-sg-accounting` for GST, invoicing, payroll, ACRA, IRAS, and retention rules.
- Use a matching built-in d6e SaaS skill when one is available. Otherwise use `d6e-effect` and `d6e-workflow` with an approved connector.
- Do not copy credentials or tokens into ERP tables, STF code, payload archives, or logs.
- Track external synchronization separately from accounting posting. A successful submission or API response is not proof that a journal was posted, and a posted journal does not prove external acceptance.

## Confirm before implementation

- provider, external organization or account, and corresponding d6e legal entity
- source of truth and direction for each object type
- provider sandbox or certification requirements and permitted external writes
- initial history window, incremental checkpoint, page or cursor behavior, and rate limits
- identifiers used by each service, including UEN, GST registration number, Peppol ID, bank account ID, filing reference, and provider object ID
- correction, cancellation, rejection, and resubmission behavior
- preparer, approver, submitter, and reconciliation responsibilities

Never infer that identifiers from different authorities or providers are interchangeable.

## d6e SQL synchronization model

Store connection metadata, object mappings, sync runs, item results, outbound work, and inbound events in short d6e SQL table names. Keep credentials outside these tables.

Track at least:

- provider, connection key, entity ID, and external organization ID
- object type, external ID, internal table and ID, external version or digest
- direction, checkpoint, start and finish times, counts, and status
- per-item source key, payload digest, result, error category, and retry count
- outbound idempotency key, request version, provider response ID, and acknowledgement time
- inbound event ID, received time, digest, and processing status

Scope external-ID uniqueness by provider, connection, and object type. Never treat an external ID alone as globally unique.

Enforce scoped SQL uniqueness for provider, connection, and webhook event ID; provider, connection, and outbox idempotency key; and sync run, source key, and source version or digest. Use an STF conditional update to claim or lease a pending row before processing so concurrent Workflows cannot execute the same inbox or outbox item.

## Integration workflow

### Inbound

1. Fetch only the declared period, page, or cursor through a SaaS skill or Effect. Prefer a provider snapshot cursor or a frozen upper watermark with stable `(updated_at, external_id)` ordering.
2. Record the source endpoint, retrieval time, and response digest before transforming data.
3. Validate mappings, types, currency, GST classification, duplicates, and closed periods in an STF.
4. Route journal creation through the controlled import-posting STF. Do not write provider responses directly to `journal_lines`.
5. Advance the fetch checkpoint only after every page and item is durably recorded. Keep fetch progress separate from processing completion, retain failed items for retry, and do not mark the run complete while unresolved failures remain.

For an API limited to mutable offset pagination, deduplication cannot recover records skipped when pages reorder. Require an overlapping rescan plus period, count, and amount reconciliation, and report the residual omission risk.

### Outbound

1. In one STF transaction, finalize the business object and create an outbox row with an immutable idempotency key.
2. Submit through a Workflow and SaaS proxy or Effect.
3. Record the response ID, external version, acknowledgement, and error through another STF.
4. Retry the same outbox row and idempotency key. Never regenerate the invoice or journal merely because transmission failed.

Record whether each endpoint accepts and enforces idempotency keys. Enable automatic retry only when that guarantee exists. Otherwise mark a timeout or unknown outcome as `indeterminate`, read back or reconcile using a deterministic external reference, and do not resend until the outcome is safely resolved.

Require user authorization before external writes, cancellations, or submissions. Represent changes affecting posted journals with reversals and replacement mappings rather than physical deletion.

## References

- InvoiceNow and access-point integration: [references/invoicenow.md](references/invoicenow.md)
- Bank statement import, matching, and reconciliation: [references/banking.md](references/banking.md)
- ACRA, IRAS, and other government submissions: [references/government-services.md](references/government-services.md)

Read only the references relevant to the request.

## Verification

- Replaying a page, webhook, or outbox row does not duplicate objects or journals.
- Missing account, partner, GST, or identifier mappings fail closed.
- Provider totals reconcile to d6e by entity, period, currency, and classification.
- Processing can resume after partial failure, timeout, delayed response, or rate limiting.
- Report sandbox-tested, read-tested, write-tested, authority-accepted, and accounting-reconciled states separately.
