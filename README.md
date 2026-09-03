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
│   └── erp-jp-integrations/  # planned
├── sg/
│   ├── erp-sg-accounting/    # implemented
│   └── erp-sg-integrations/  # planned
└── migration/
    └── erp-migrate-trebit/   # planned
```

The initial implementation units are `erp-core`, `erp-jp-accounting`, and `erp-sg-accounting`. Add the integration and legacy `trebit` migration directories only when their content is implemented; do not create empty placeholders.
