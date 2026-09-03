# STF and Workflow patterns

## Use a single STF for atomic accounting changes

Keep posting, reversal, settlement, period close, and similar operations in one STF when all SQL statements must either succeed or fail together. Calls to `sql()` inside one STF share one database transaction.

Example posting-STF responsibilities:

1. Validate UUID, date, enum, and monetary input formats.
2. Check for a duplicate source key.
3. Confirm that the accounting period is open.
4. Confirm that accounts, currencies, and tax classifications are effective.
5. Sum debits and credits in transaction and functional currencies and require balance.
6. Write the header and every line.
7. Return the journal ID and independently verifiable totals.

## Use Workflow for orchestration

Use a Workflow only when the process must:

- Import a file, validate it, post it, and generate an output file in sequence.
- Fetch from an external API, transform the result with an STF, and send it to another API.
- Reuse multiple steps containing human input or approval.
- Run multiple independent STFs in a defined order.

Each STF in a Workflow uses a separate transaction. A later failure does not automatically roll back an earlier step. Make each step idempotent and retryable through explicit state and source keys. Define a compensating STF when necessary.

## Versioning

- Pin STF and Effect versions in a Workflow when statutory calculations, close, or posting must be reproducible.
- When following the latest version, run regression tests after a change and record when the new logic begins to apply.
- Do not embed time-dependent values such as tax rates in STF code. Read them from an effective-dated SQL master.

## Input safety

STF `sql()` does not accept a separate parameter argument. Therefore:

- Strictly validate UUIDs, ISO dates, monetary values, currency codes, and states.
- Never construct SQL identifiers from user input. Use a fixed mapping.
- Escape string values as PostgreSQL literals by doubling single quotes.
- Do not return an error response as a normal success value. Throw on an accounting-invariant violation so the transaction fails.
- Do not catch an exception after a write and return a normal value such as `{ success: false }`. That would mark the STF as successful and may commit partial writes. Perform only necessary cleanup and rethrow.

## Validation before saving

Before creating a saved STF, pass the same code to `d6e_instant_run_stf`. Test an allowed case and rejection of unbalanced data, duplicates, invalid dates, closed periods, and insufficient authorization before calling `d6e_create_stf`.
