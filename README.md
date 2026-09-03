# erp-skills

`erp-skills` は、[d6e](https://github.com/d6e-ai/d6e) を基盤として日本およびシンガポール向けの基幹システムを設計・構築するための Agent Skills リポジトリです。

これらのスキルは、d6e のワークスペース分離された PostgreSQL、SQL 実行、STF、Workflow、Policy、外部サービス連携を利用することを前提とします。独立した ERP データベースや、旧 `trebit` の Git/YAML 台帳を再実装するためのものではありません。

現在の設計方針は [docs/design.md](docs/design.md) を参照してください。

## Planned layout

```text
skills/
├── shared/
│   └── erp-core/
├── jp/
│   ├── erp-jp-accounting/
│   └── erp-jp-integrations/
├── sg/
│   ├── erp-sg-accounting/
│   └── erp-sg-integrations/
└── migration/
    └── erp-migrate-trebit/
```

各ディレクトリは実際に内容を実装するときに追加します。空のプレースホルダーは作成しません。
