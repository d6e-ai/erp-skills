# Singapore bank integrations

Confirm the actual bank API, aggregator, file format, or payment provider before implementation. Do not invent endpoints, authentication behavior, or statement fields.

## Connection boundary

- Keep credentials, consent artifacts, and refresh tokens in the approved d6e or provider credential store, not ERP SQL tables.
- Identify bank accounts, virtual accounts, cards, payment processors, and wallets as separate connections.
- Do not assume that PayNow, FAST, card-settlement, or gateway references are bank-statement transaction IDs.
- Use a matching d6e SaaS skill when available; otherwise use an authenticated Effect or an approved file import.

## SQL model

Use short tables such as `bank_stmts` and `bank_lines`, or equivalent existing tables, to retain:

- entity ID, internal bank account ID, provider, and external account ID
- statement ID, period, opening balance, closing balance, and currency
- provider transaction ID, transaction date, value date, amount, and direction
- description, counterparty, payment reference, and raw payload digest
- source file ID or sync run ID, retrieval time, and mapping version
- match status, matched payment ID, matched journal ID, and reviewer

Use exact decimal values and document each provider's sign convention before normalization. If transaction IDs are not stable, retain a secondary deduplication key based on account, date, amount, reference, and raw digest.

## Matching workflow

1. Prefer exact payment, invoice, virtual-account, or remittance references.
2. Generate candidates using amount, currency, date, and counterparty.
3. Send ambiguous, split, combined, fee-netted, or foreign-exchange settlements for review.
4. Send only approved matches to the settlement STF.

Treat fuzzy matching as a suggestion, not an accounting fact. Keep unmatched, candidate, reconciled, and excluded states separate. A bank line is evidence of movement; it is not automatically a journal entry.

## Reconciliation

- Verify opening balance plus movements equals closing balance.
- Classify every statement line as matched, approved exclusion, or unresolved.
- Compare the reconciled closing balance to the bank control account.
- Link correction, duplicate, and reversal lines to their originals.
- Confirm that file re-import and API overlap do not duplicate lines.

Never hide a residual difference with an automatic plug entry. Report the amount, currency, line IDs, candidates, and reason.
