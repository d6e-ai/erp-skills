# d6e implementation boundary

Verified against the local d6e source on 2026-09-03.

## SQL execution

- `d6e_sql` rewrites unqualified workspace table names to `user_data.ws_{workspace_id}_...`. Do not write the prefix manually.
- The workspace prefix is 40 characters and PostgreSQL identifiers are limited to 63 characters, so user table names must not exceed 23 characters. Start a name with a lowercase letter; use only lowercase letters, digits, and single underscores. Do not use a `pg_` prefix, trailing underscore, consecutive underscores, or SQL reserved word.
- Each `d6e_sql` call may contain only one SQL statement.
- The reviewed implementation supports `CREATE TABLE`, supported forms of `ALTER TABLE`, `DROP TABLE`, `SELECT`, `INSERT`, `UPDATE`, and `DELETE`.
- The reviewed implementation does not accept `CREATE INDEX`. At a scale that requires indexes, verify support in the target d6e version or report the need for a d6e platform enhancement instead of assuming success.
- Foreign-key `ON DELETE` and `ON UPDATE` actions may use only `RESTRICT` or `NO ACTION`. Do not depend on `CASCADE`, `SET NULL`, or `SET DEFAULT`.
- DDL requires a workspace administrator or a member of `ddl_policy_group`. An STF cannot execute DDL through `sql()`.
- DML is subject to default-deny Policy enforcement. Creating a table does not make it usable.
- Do not assume that every table name in a CTE, set operation such as `UNION`, or expression subquery will be rewritten to a workspace name. Prefer simple statements for accounting operations. When a complex statement is necessary, validate `executed_sql` and denied cases on the target d6e version first.

## Columns

Some d6e documentation describes automatically added system columns, but the reviewed SQL transformation code does not inject columns. Define required fields explicitly in DDL, for example:

```sql
id UUID PRIMARY KEY DEFAULT uuidv7(),
created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
deleted_at TIMESTAMPTZ
```

Inspect `executed_sql` and the actual table on the target instance. If behavior differs, trust the execution result.

Do not assume that a trigger updates `updated_at`. Set `updated_at = now()` explicitly in update SQL.

## STF runtime

- Write STF code at the top level without wrapping it in a function declaration, and finish with `return`.
- `$input` is the input, `$caller` is the authenticated user ID or `null`, and `$sources` contains Workflow input steps.
- `sql(query)` accepts one argument. `SELECT` returns an array of rows; DML returns an affected-row count.
- SQL calls inside one STF share one database transaction. Keep posting validation and all related writes in the same STF.
- A saved STF receives SQL permission through Policies for the STF subject. Instant execution uses the calling user's Policies.
- Never interpolate input directly into SQL. Validate UUIDs, dates, enum values, and monetary formats; escape string literals using PostgreSQL rules; select identifiers only from a fixed allowlist.
- `NUMERIC` remains exact inside SQL, but results pass through JSON. Cast monetary values to text, such as `amount::text`, before returning them to an STF, and do not convert them to JavaScript `Number` values.

## Workflow runtime

- Do not wrap work that fits in one QJS STF inside a Workflow.
- Each STF in a Workflow runs in a separate transaction. Do not assume an atomic commit across multiple STFs.
- Pin STF and Effect versions in a Workflow when audit reproducibility requires it. Otherwise, state the impact of following the latest version.
- Field-mapping variable paths begin with `$input`, `$sources`, or `$steps[n]`.

## Verification

Before reporting a completed implementation, verify with actual tool results that:

1. Tables and constraints exist.
2. An intended subject can perform each allowed operation.
3. An out-of-scope subject is denied.
4. A valid balanced journal can be posted.
5. An unbalanced journal, closed period, and duplicate source key are rejected.
6. Balances and report aggregates match a known numerical example.
