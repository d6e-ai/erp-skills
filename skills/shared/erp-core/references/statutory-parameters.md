# Statutory parameter management

Separate statutory rates, recoverable proportions, monetary thresholds, filing or issue deadlines, payroll rate tables, and form or taxonomy versions from business logic and regulatory prose.

## Two layers

1. Each country's `references/parameter-inventory.md` is a seed inventory for detecting and reviewing official changes before loading them into d6e.
2. A short d6e SQL table such as `law_params` is the runtime system of record read by STFs.

Do not treat Markdown as the runtime source of truth or assume that an STF can read the repository. Conversely, do not update only d6e and lose the source and review history.

## Inventory contract

Every entry must contain at least the following fields. A country inventory must be a machine-processable table with the same fields. If presentation splits the entry across two tables, `key` and `version` must join them uniquely.

| Field | Meaning |
| --- | --- |
| `key` | Stable ID containing country, regime, and use; it does not change across generations of the same parameter |
| `version` | Version within the same key and scope; do not embed suffixes such as `v1` in `key` |
| `value_type` | A type such as `numeric`, `date`, `version`, or `dataset` |
| `value` | Decimal, integer, date, version, or dataset identifier matching `value_type` |
| `unit` | A unit such as `ratio`, `JPY`, `SGD`, `days`, `years`, `date`, or `version` |
| `scope_key` / `scope` | Stable machine-readable scope ID and a description of transactions, entities, employee classes, and exceptions |
| `anchor_type` | Date used to evaluate applicability, such as `transaction_date`, `tax_period_start`, `pay_date`, or `filing_date` |
| `effective_from` / `effective_to` | ISO DATE applicability interval, inclusive at the start and exclusive at the end; leave blank only with `verify` status |
| `status` | `active`, `future`, `expired`, or `verify` |
| `approval` | `pending`, `approved`, or `rejected`; repository inventory entries remain `pending` until independently approved |
| `source` | Primary government or regulator URL |
| `checked_on` | Date on which a human or agent reviewed the source |
| `supersedes` | Previous entry in `key@version` form |
| `consumers` | STFs, reports, Workflows, and validation cases that use the entry |

Do not mix `10`, `10%`, and `0.10` as representations of one ratio. Use a normalized SQL `NUMERIC` ratio such as `0.10`. Do not combine a scalar value and its applicability conditions in one free-text value. Represent complex CPF rate tables, phased rules, and taxonomies with a stable dataset key, a dataset version, and typed detail rows instead of reducing them to one scalar.

## d6e SQL model

At runtime, `law_params` stores at least `jurisdiction`, `param_key`, `scope_key`, `version`, `value_type`, a typed value, `unit`, `anchor_type`, a half-open effective interval, `approval_status`, source URL, and review date. Separate typed values into columns such as `num_value`, `date_value`, `text_value`, and `dataset_key`, and use a `CHECK` constraint to require exactly the column that matches `value_type`. Reject overlapping periods for the same key, scope, and anchor through an available d6e SQL constraint or validation inside the approval STF. Never overwrite a historical row.

An STF must receive `anchor_type` and `as_of_date` from its caller and resolve exactly one approved row. Do not substitute a transaction date for a tax-period start date, payment date, filing date, incorporation date, or another statutory anchor. Stop when there is no match, multiple matches, or only unapproved or `verify` rows. Do not select `future` rows before their effective date. Permit `expired` rows when reproducing a historical date. Select the row applicable to the transaction, not simply the latest row.

`tax_codes` is a transaction-classification master, not the system of record for statutory rates. It normally references an approved `law_params` row. When a transaction or tax calculation snapshots a rate for audit reproducibility, store the referenced `law_params` ID and version, and do not treat that snapshot as the authoritative input for a later calculation.

## Amendment workflow

1. Confirm enactment, commencement, and transitional provisions in an official primary source.
2. Preserve the stable key, add a new version to the country inventory, set the old row's exclusive end date, and record `supersedes`.
3. Add a future value to d6e SQL with `future` status, but do not select it before its effective date.
4. Run regression cases around the boundary, including returns, cancellations, retrospective processing, and unresolved conditions.
5. After independent approval, mark the d6e row `approved` and record the affected STF and report versions.
6. Confirm that historical transactions still reproduce the old value after commencement.

When a reform document is published but enactment, commencement, or company-specific scope remains uncertain, keep the entry in `verify` status. An agent must not activate a production value by itself.
