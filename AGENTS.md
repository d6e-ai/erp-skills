# Repository instructions

Read `docs/design.md` before working in this repository. When the design changes, update the accepted current state in `docs/design.md` and keep discussions and dated decision records under `docs/meeting-notes/`.

## Language

- Use English for repository instructions, general documentation, shared skills, Singapore skills, migration skills, and future integration skills.
- Japanese content is allowed only under `skills/jp/`, where Japanese improves the accuracy and usability of Japan-specific accounting and statutory guidance.
- Keep code identifiers, SQL names, product names, and official terminology unchanged when translation would reduce precision.

## Platform boundary

- The skills in this repository build ERP systems on d6e.
- Use d6e `d6e_sql` as the primary path for data definition, storage, retrieval, and aggregation, and follow the built-in `d6e-sql` skill when writing SQL.
- Use d6e STF for transactional business logic, Workflow for multi-step processing, and Policy for authorization.
- Do not introduce a database, ORM, API server, or ledger engine outside d6e without an explicit requirement.
- Do not make the `trebit` Git/YAML write model, CLI, REST API, or branch management part of the new system. Isolate only the migration knowledge that remains necessary in `erp-migrate-trebit`.

## Skill organization

- Use `skills/shared/` for country-neutral guidance.
- Store country-specific skills under `skills/jp/` and `skills/sg/`.
- Begin each country with broad skills corresponding to `accounting` and `integrations`. Split tax, invoicing, payroll, statutory reporting, and similar details into `references/` first, and create another skill only when the area needs an independent discovery or execution unit.
- Keep skill names and directory names identical. Use the naming families `erp-core`, `erp-jp-*`, `erp-sg-*`, and `erp-migrate-*`.
- Do not use the `d6e-` prefix, which is reserved for built-in d6e skills.
- Keep decision procedures, important constraints, and reference routing concise in `SKILL.md`. Put detailed regulations, data models, SQL patterns, and acceptance criteria in adjacent `references/` files.
- Keep a reference file inside the skill that uses it. Do not create implicit dependencies on repository-wide shared references.
- Do not add empty directories, placeholder examples, or speculative assets.

## d6e implementation constraints

- Keep SQL table names short enough for the PostgreSQL identifier limit after d6e adds its workspace prefix.
- Do not assume that d6e automatically adds system columns. Define required columns such as `id`, `created_at`, `updated_at`, and `deleted_at` explicitly after checking the target instance behavior.
- Design for d6e default-deny Policy enforcement and state the permitted operation and subject explicitly.
- Treat double-entry balance, period close, correction, and audit trails as SQL schema and STF invariants.
- Do not scatter statutory rates, thresholds, deadlines, rate-table versions, or form versions through prose or STF code. Collect stable keys, versions, date anchors, effective periods, sources, review dates, and approval states in each country's `references/parameter-inventory.md`. Use d6e SQL `law_params` as the runtime system of record. `tax_codes` holds classifications and references, not an independent authoritative rate.
- Do not rely on arbitrary update SQL issued from a conversation for recurring business operations. Define reusable updates as STFs or Workflows with validation and authorization.
- Reuse existing built-in d6e SaaS skills for external accounting-service API details. Keep this repository focused on ERP mappings and business workflows.
- d6e does not install skill-local `scripts/` as runtime resources. Express required runtime behavior in SQL, STF, or Workflow instead of depending on local scripts.
