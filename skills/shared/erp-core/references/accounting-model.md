# Core accounting model

This model is a foundation for country-specific requirements, not a completed statutory chart of accounts.

## Recommended tables

| Table | Purpose | Important keys |
| --- | --- | --- |
| `legal_entities` | Legal entity or ledger unit | `id`, `code`, `base_currency` |
| `currencies` | Currency and precision | `code`, `minor_units` |
| `fiscal_periods` | Accounting periods and close state | `entity_id`, `start_date`, `end_date`, `status` |
| `accounts` | Chart of accounts | `entity_id`, `code`, `account_type`, `normal_side` |
| `partners` | Customers, suppliers, and similar parties | `entity_id`, `code`, tax registration fields |
| `tax_codes` | Transaction tax-classification master | `jurisdiction`, `code`, classification, `law_param_id` |
| `journals` | Journal headers | `entity_id`, `journal_no`, `status`, source identity |
| `journal_lines` | Debit and credit lines | `journal_id`, `line_no`, `account_id`, `side`, `amount` |
| `doc_links` | Source-document relationships | `document_type`, `document_id`, `storage_file_id` |

Every name is 23 characters or fewer. Check existing tables for overlap before adding another physical table.

Use `law_params` as the system of record for statutory rates and recoverable proportions. Do not duplicate an independent authoritative value in `tax_codes`. When a calculation snapshots a rate, also store the ID and version of the `law_params` row used.

## Header and line rules

Store at least the following on `journals`:

- Legal entity, journal number, journal date, and accounting period
- A state such as `draft`, `posted`, or `reversed`
- Source system, source type, and source key
- Transaction and functional currencies
- Poster, posting time, and reversed journal
- Description and creation or update timestamps

Store at least the following on `journal_lines`:

- Journal, line number, and account
- `debit` or `credit`
- A positive transaction-currency amount
- Functional-currency amount and applied exchange rate
- Analysis dimensions such as tax classification, partner, and department

Do not mix positive and negative values in line amounts. Store a positive amount and express direction with `side`. Reporting queries may convert this to one consistent sign convention, such as debit positive and credit negative.

Use `NUMERIC` columns with defined precision. When SQL returns an amount or exchange rate to an STF, cast it to text with expressions such as `amount::text` instead of converting it to a JavaScript floating-point number. Integer minor units are also valid where the currency scale is fixed, but record the scale and rounding rules for each currency.

## Constraints

Enforce every suitable invariant with SQL constraints.

- Make account codes and journal numbers unique within an entity.
- Add a unique constraint on `journal_lines(journal_id, line_no)`.
- Restrict states, account types, normal balances, and line sides with `CHECK` constraints.
- Require `amount > 0` and `fx_rate > 0`.
- Make each source-system and source-key pair unique.
- Use foreign keys between entities, periods, accounts, tax classifications, and journals.

Journal-wide balance and closed-period checks span multiple rows, so enforce them in the posting STF. Do not rely only on application code; keep one controlled posting path.

## State transitions

```text
draft -> posted -> reversed
```

- `draft`: Editable and excluded from ledger balances.
- `posted`: Immutable and included in ledger balances.
- `reversed`: The original remains stored and references its reversing journal.

After a reversal, post corrected content as a separate new journal. Do not overwrite historical facts.

## Reporting

Build a trial balance from posted lines only, aggregating debit, credit, and net amounts by account. Derive profit-or-loss and financial-position classification from the account master's `account_type`. Keep mappings to country-specific presentation lines in the country skill.

At close, reconcile at least the following:

- Total debits in all posted journals equal total credits.
- Subledger totals equal general-ledger control accounts.
- Receivable and payable balances equal total open items.
- Tax control accounts equal totals by tax classification.
- Opening balance plus period movement equals closing balance.
