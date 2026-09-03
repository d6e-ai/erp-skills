# Singapore GST

Official information reviewed: 2026-09-03. Re-check rates, thresholds, transitional rules, and scope before production use.

## Current baseline

Refer to [parameter-inventory.md](parameter-inventory.md) for the current GST rate and its effective date. Do not determine the applicable rate mechanically from the transaction date alone; consider the time of supply, transitional rules, supply classification, and registration status.

Official sources:

- [IRAS: Prevailing rate of 9%](https://www.iras.gov.sg/taxes/goods-services-tax-%28gst%29/charging-gst-%28output-tax%29/when-to-charge-goods-and-services-tax-%28gst%29/prevailing-rate-of-9)
- [IRAS: Overview of GST rate change](https://www.iras.gov.sg/taxes/goods-services-tax-%28gst%29/gst-rate-change/gst-rate-change-for-business/overview-of-gst-rate-change)
- [IRAS: Invoicing, price display and record keeping](https://www.iras.gov.sg/taxes/goods-services-tax-%28gst%29/basics-of-gst/invoicing-price-display-and-record-keeping)

## SQL data requirements

Store the following in `tax_codes` or an equivalent table:

- `jurisdiction = 'SG'`
- classification such as standard-rated, zero-rated, exempt, or out-of-scope
- direction such as output tax, input tax, or reverse charge
- a `law_params` reference that resolves the rate and recoverable proportion, plus a rule ID that identifies the applicable conditions
- `effective_from`, `effective_to`, source URL, and verification date
- rounding method and currency-conversion rule ID

Do not change a rate or threshold by overwriting a historical row. When snapshotting a rate on a transaction, also store the ID and version of the referenced `law_params` row.

## Transaction requirements

Make every GST aggregation traceable to:

- supplier or customer and GST registration number
- supply date, invoice date, posting date, and accounting period
- supply classification, exclusive amount, GST amount, and inclusive amount
- transaction-currency amounts and SGD amounts for GST purposes
- exchange rate, rate date, and rate source
- tax invoice, credit or debit note, receipt, and import evidence
- input-tax claim status and review reason

## Calculation control

IRAS describes both summing GST calculated for individual line items and applying the rate to the total amount excluding tax. Apply the selected method and rounding consistently, and store them on the invoice.

The tax-calculation STF must select a valid tax code from SQL and return the rate used, exclusive amount, GST amount, inclusive amount, and rounding delta. Never default an unknown classification or claim eligibility to standard-rated or claimable.
