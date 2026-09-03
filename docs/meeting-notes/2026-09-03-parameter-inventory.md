# Statutory parameter inventory

Date: 2026-09-03

## Decision

- Do not scatter values that change with legislation, such as statutory rates, recoverable proportions, monetary thresholds, deadlines, rate tables, and form versions, through prose or STF code.
- Keep `references/parameter-inventory.md` in the Japan and Singapore accounting skills, collecting values, units, effective periods, states, primary sources, and review dates.
- Use repository inventories for change detection, review, and initial seeding. Use an effective-dated d6e SQL master as the runtime system of record.
- Future values may be registered with `future` status, but an STF must not select them until both approval and the effective date permit it.
- After an amendment, add a new period row rather than overwriting an existing row, and run regression tests around the effective-date boundary.
- Keep each key stable across generations. Store the version and half-open effective interval in separate fields instead of embedding the version in the key.
- Pass an `anchor_type` that distinguishes transaction dates, tax-period start dates, payment dates, filing dates, and other applicability dates.
- Country inventories must carry every field in the shared contract and separate scalar values, dataset references, and typed dataset rows.
- Use `law_params` as the authoritative source for statutory rates. `tax_codes` holds classifications and references.

[`docs/design.md`](../design.md) is the normative source for this policy.
