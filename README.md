# erp-skills

`erp-skills` は、[d6e](https://github.com/d6e-ai/d6e) を基盤として日本およびシンガポール向けの基幹システムを設計・構築するための Agent Skills リポジトリです。

これらのスキルは、d6e のワークスペース分離された PostgreSQL、SQL 実行、STF、Workflow、Policy、外部サービス連携を利用することを前提とします。独立した ERP データベースや、旧 `trebit` の Git/YAML 台帳を再実装するためのものではありません。

現在の設計方針は [docs/design.md](docs/design.md) を参照してください。

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

`erp-core`、`erp-jp-accounting`、`erp-sg-accounting`を最初の実装単位とします。`integrations`と旧`trebit`移行は、内容を実装するときにディレクトリを追加し、空のプレースホルダーは作成しません。
