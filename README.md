# erp-skills

`erp-skills` is an Agent Skills repository for designing and building Japan and Singapore ERP systems on [d6e](https://github.com/d6e-ai/d6e).

These skills assume d6e workspace-isolated PostgreSQL, SQL execution, STF, Workflow, Policy, and external-service integrations. They do not reimplement an independent ERP database or the legacy `trebit` Git/YAML ledger.

See [docs/design.md](docs/design.md) for the accepted design.

Repository documentation is written in English by default. Japan-specific skill content under `skills/jp/` may be written in Japanese.

## Current layout

```text
skills/
├── shared/
│   └── erp-core/             # implemented
├── jp/
│   ├── erp-jp-accounting/    # implemented
│   └── erp-jp-integrations/  # implemented
└── sg/
    ├── erp-sg-accounting/    # implemented
    └── erp-sg-integrations/  # implemented
```

The repository implements the shared ERP core plus country accounting and integration units. Valid accounting concepts from Trebit are carried forward in `erp-core`; no Trebit-specific runtime or migration skill is retained. Continue to add files only when they contain usable guidance; do not create empty placeholders.
