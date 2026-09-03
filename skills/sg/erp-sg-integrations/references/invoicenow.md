# InvoiceNow integration

Official starting points reviewed: 2026-09-03. Re-check the applicable requirement, access-point specification, and provider contract before implementation.

- [IRAS: GST InvoiceNow Requirement](https://www.iras.gov.sg/taxes/goods-services-tax-%28gst%29/gst-invoicenow-requirement)
- [IMDA: InvoiceNow](https://www.imda.gov.sg/invoicenow)

## Responsibility boundary

- Use the approved InvoiceNow solution provider or Peppol Access Point selected by the entity. Do not assume d6e has a direct IRAS or Peppol connector.
- Use `erp-sg-accounting` and approved d6e `law_params` rows to determine obligation and applicable rules. Do not duplicate phase dates or thresholds here.
- Keep UEN, GST registration number, Peppol participant ID, access-point account ID, document ID, and provider message ID as distinct fields.
- Keep d6e invoice status, accounting-posting status, transmission status, network acknowledgement, and IRAS status separate.

## Outbound contract

Before creating an outbox row, validate:

- supplier and customer legal identifiers and addresses
- invoice number, issue date, currency, payment terms, and document type
- line quantities, unit prices, allowances or charges, GST classifications, and totals
- SGD tax amounts and exchange-rate evidence when required
- reference to the original invoice for a credit or debit note
- selected provider, access-point routing identifier, and schema version

Store the canonical d6e invoice ID, immutable payload digest, schema version, mapping version, and idempotency key. Generate a new payload version only when the source invoice is validly corrected; do not mutate the payload associated with an earlier submission attempt.

## Submission states

Use explicit states such as:

```text
pending -> submitted -> acknowledged
                   \-> rejected
pending -> failed_retryable
pending -> failed_terminal
```

A provider HTTP success can mean only that the request was received. Preserve all later network or authority acknowledgements and rejection details. Link retries to the original outbox row and link resubmissions after correction to the rejected document.

## Inbound documents

For received invoices, deduplicate using provider, connection, sender participant ID, document ID, and payload digest. Store the original payload or file in d6e storage with a digest, then map it to a draft payable. Do not post automatically when supplier, GST classification, account, currency conversion, or duplicate status is unresolved.

## Reconciliation

- Every eligible d6e invoice has one current submission outcome or a documented exclusion.
- Every provider message ID maps to one outbox version.
- Accepted totals match the source invoice by currency and GST classification.
- Credit and debit notes link to the original invoice and journal.
- Rejected, retryable, and unresolved documents remain visible and do not advance a completion checkpoint.
