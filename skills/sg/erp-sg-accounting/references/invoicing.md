# Singapore invoicing and InvoiceNow

Official information reviewed: 2026-09-03. InvoiceNow implementation phases can change and must be re-checked.

## Tax invoices

Refer to [parameter-inventory.md](parameter-inventory.md) for issuance deadlines, the simplified-tax-invoice limit, and the InvoiceNow phase schedule. For a standard-rated supply to a GST-registered customer, retain at least the following on the tax invoice:

- the words "Tax Invoice"
- supplier name and address, and GST registration number
- invoice date and unique invoice number
- customer name and address
- description of goods or services
- GST rate
- total excluding GST, total GST, and total including GST
- gross amounts classified as exempt, zero-rated, or other supplies

For an invoice not denominated in SGD, convert at least the total excluding tax, total including tax, and GST payable to SGD, and track the approved rate source.

Official source:

- [IRAS: Invoicing customers](https://www.iras.gov.sg/taxes/goods-services-tax-%28gst%29/basics-of-gst/invoicing-price-display-and-record-keeping/invoicing-customers)

## InvoiceNow

InvoiceNow is Singapore's e-invoicing network based on the Peppol standard. Requirements to submit GST invoice data to IRAS are being introduced in phases. Do not hard-code applicable dates. Store each entity's phase, obligation start date, exemption, Peppol ID, and provider.

Official sources:

- [IRAS: GST InvoiceNow Requirement](https://www.iras.gov.sg/taxes/goods-services-tax-%28gst%29/gst-invoicenow-requirement)
- [IMDA: InvoiceNow](https://www.imda.gov.sg/invoicenow)

## d6e workflow

1. Finalize the invoice and tax totals in SQL.
2. Use an STF to validate required fields, GST classification, SGD conversion, and rounding.
3. Create the InvoiceNow payload as an outbound record separate from the ledger.
4. Submit through an Effect or external provider, and store the provider message ID and response.
5. Reuse the same idempotency key for retries, and do not regenerate the journal entry.
6. Link each credit or debit note to the original invoice and original journal.

Track submission status and accounting-posting status independently. Do not treat successful submission as evidence of successful accounting posting.
