# ERP authorization on d6e

## Roles are duties, not screens

Design authorization around accounting duties a subject may perform, not around screens. Typical duties include:

| Duty | Typical access |
| --- | --- |
| Master data maintainer | Read, create, and narrowly update master data |
| Preparer | Create draft documents and update their own drafts |
| Approver | Read and approve drafts, but not approve their own work |
| Poster STF | Update only the tables required to post approved documents |
| Accountant | Read the ledger, reconcile balances, and close periods |
| Auditor | Read all periods and audit logs, with no write access |
| Integration STF | Access only the import tables and document creation needed for its assigned source system |

Duties may be combined in a small organization, but the system must still preserve auditable preparer, approver, and poster identities.

## d6e Policy mapping

1. Add a user or STF to a Policy Group.
2. Define Policies per table and separately for `select`, `insert`, `update`, and `delete`.
3. Use legal-entity IDs, states, owners, and similar fields in conditional Policies so other entities and posted records remain out of scope.
4. Give each saved STF that writes data a dedicated Policy Group.
5. Separate DDL permission from normal operation and minimize membership in `ddl_policy_group`.

DML without a Policy is denied. An operation succeeding as an administrator does not prove that it will succeed for a normal user or STF.

Pass conditions as JSON objects, not SQL strings. The reviewed current API implementation supports `$eq`, `$ne`, `$gt`, `$gte`, `$lt`, `$lte`, `$in`, and `$null`; refer to the current subject ID with the string `"$user_id"`. If tool documentation differs, follow the target d6e version's API implementation and observed execution results.

## Current enforcement limits

The implementation reviewed on 2026-09-03 has constraints that materially affect accounting authorization:

- A conditional `insert` Policy participates in membership authorization, but its condition is not applied to the inserted row. Validate insert conditions such as legal entity, state, and source system in a saved STF, and do not grant normal users direct `insert` access to the target table.
- For DML containing a JOIN, d6e builds one Policy WHERE clause around the first parsed table. Do not create an authorization boundary by joining sensitive tables; use separately authorized single-table reads or a dedicated STF.
- Do not use complex CTEs, set operations, or subqueries as an authorization boundary. Run both allowed and denied cases before adopting a design.

The platform may improve these limits. If target-version behavior differs, trust the observed validation result and current d6e specification.

## Accounting protections

- Do not allow users to directly `update` or `delete` posted journals.
- Only the posting STF may transition `journals.status` to `posted` and finalize its lines.
- Use separate permissions for period close and reopen.
- When self-approval is forbidden, compare preparer and approver IDs in the STF.
- Validate that an integration STF cannot create source keys for another source system.
- Give auditors read-only access to accounting tables and audit logs.

## Verification matrix

Run denied cases as well as allowed cases for every subject.

| Subject | Allowed test | Denied test |
| --- | --- | --- |
| Preparer | Create a draft | Update a posted journal |
| Approver | Approve another user's draft | Self-approve |
| Poster STF | Post an approved document | Post an unapproved document or into a closed period |
| Auditor | Read all records | Insert, update, or delete |
| Integration STF | Import from its assigned source | Spoof another source or import a duplicate |
