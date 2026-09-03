# 日本 法定パラメーター・インベントリ

Checked on: 2026-09-03. このファイルは変更把握とd6e SQL投入レビューのためのseedであり、実行時の正本ではない。repository上の行はすべて独立承認前の`pending`である。

Statusは2026-09-03時点。`active`でも取引条件を満たすとは限らず、scalarだけで税務判定しない。空の適用日は未確認を意味し、`verify`以外では許可しない。

## Parameter values

`effective_from`を含み、`effective_to`を含まない。適用期間は`anchor_type`が示す日付に対して評価する。

| Key | Version | Type | Value | Unit | Scope key | Anchor type | Effective from | Effective to |
| --- | --- | --- | ---: | --- | --- | --- | --- | --- |
| `jp.ct.standard.total` | `2019.1` | numeric | 0.10 | ratio | `standard_supply` | `transaction_date` | 2019-10-01 | |
| `jp.ct.standard.national` | `2019.1` | numeric | 0.078 | ratio | `standard_supply` | `transaction_date` | 2019-10-01 | |
| `jp.ct.standard.local` | `2019.1` | numeric | 0.022 | ratio | `standard_supply` | `transaction_date` | 2019-10-01 | |
| `jp.ct.reduced.total` | `2019.1` | numeric | 0.08 | ratio | `reduced_supply` | `transaction_date` | 2019-10-01 | |
| `jp.ct.reduced.national` | `2019.1` | numeric | 0.0624 | ratio | `reduced_supply` | `transaction_date` | 2019-10-01 | |
| `jp.ct.reduced.local` | `2019.1` | numeric | 0.0176 | ratio | `reduced_supply` | `transaction_date` | 2019-10-01 | |
| `jp.invoice.start_date` | `2023.1` | date | 2023-10-01 | date | `qualified_invoice` | `transaction_date` | 2023-10-01 | |
| `jp.invoice.nonreg_credit` | `2023.1` | numeric | 0.80 | ratio | `nonregistered_purchase` | `transaction_date` | 2023-10-01 | 2026-10-01 |
| `jp.invoice.nonreg_credit` | `2026.1` | numeric | 0.70 | ratio | `nonregistered_purchase` | `transaction_date` | 2026-10-01 | 2028-10-01 |
| `jp.invoice.nonreg_credit` | `2028.1` | numeric | 0.50 | ratio | `nonregistered_purchase` | `transaction_date` | 2028-10-01 | 2030-10-01 |
| `jp.invoice.nonreg_credit` | `2030.1` | numeric | 0.30 | ratio | `nonregistered_purchase` | `transaction_date` | 2030-10-01 | 2031-10-01 |
| `jp.invoice.nonreg_credit` | `2031.1` | numeric | 0.00 | ratio | `nonregistered_purchase` | `transaction_date` | 2031-10-01 | |
| `jp.invoice.nonreg_cap` | `2026.1` | numeric | 100000000 | JPY | `counterparty_period_total` | `tax_period_start` | 2026-10-01 | |
| `jp.records.corp.general` | `unverified` | numeric | 7 | years | `corporate_general` | `filing_due_date` | | |
| `jp.records.corp.loss` | `unverified` | numeric | 10 | years | `qualifying_loss_year` | `filing_due_date` | | |
| `jp.payroll.withholding` | `2026.1` | dataset | `jp-withholding-2026` | dataset | `national` | `pay_date` | 2026-01-01 | 2027-01-01 |
| `jp.payroll.health_insurance` | `2026.1` | dataset | `jp-health-insurance-2026` | dataset | `kyokai_kenpo_prefecture` | `contribution_month` | 2026-03-01 | |
| `jp.payroll.employee_pension` | `unverified` | dataset | `jp-employee-pension-current` | dataset | `insured_class` | `contribution_month` | | |
| `jp.payroll.employment_insurance` | `2026.1` | dataset | `jp-employment-insurance-2026` | dataset | `business_class` | `pay_date` | 2026-04-01 | 2027-04-01 |

## Governance metadata

| Key | Version | Status | Approval | Checked on | Supersedes | Consumers | Scope | Official source |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `jp.ct.standard.total` | `2019.1` | active | pending | 2026-09-03 | | `jp_tax_calc` | 標準税率合計 | [NTA 6102][nta-6102] |
| `jp.ct.standard.national` | `2019.1` | active | pending | 2026-09-03 | | `jp_tax_calc` | 標準税率の国税分 | [NTA 6102][nta-6102] |
| `jp.ct.standard.local` | `2019.1` | active | pending | 2026-09-03 | | `jp_tax_calc` | 標準税率の地方消費税分 | [NTA 6102][nta-6102] |
| `jp.ct.reduced.total` | `2019.1` | active | pending | 2026-09-03 | | `jp_tax_calc` | 軽減対象判定は別rule | [NTA 6102][nta-6102] |
| `jp.ct.reduced.national` | `2019.1` | active | pending | 2026-09-03 | | `jp_tax_calc` | 軽減税率の国税分 | [NTA 6102][nta-6102] |
| `jp.ct.reduced.local` | `2019.1` | active | pending | 2026-09-03 | | `jp_tax_calc` | 軽減税率の地方消費税分 | [NTA 6102][nta-6102] |
| `jp.invoice.start_date` | `2023.1` | active | pending | 2026-09-03 | | `jp_invoice_validation` | 適格請求書等保存方式の開始日 | [NTA 6498][nta-6498] |
| `jp.invoice.nonreg_credit` | `2023.1` | active | pending | 2026-09-03 | | `jp_input_tax_credit` | 免税事業者等からの仕入れに係る経過措置 | [NTA 2026 reform][nta-2026] |
| `jp.invoice.nonreg_credit` | `2026.1` | future | pending | 2026-09-03 | `jp.invoice.nonreg_credit@2023.1` | `jp_input_tax_credit` | 適用条件と課税期間も確認 | [NTA 2026 reform][nta-2026] |
| `jp.invoice.nonreg_credit` | `2028.1` | future | pending | 2026-09-03 | `jp.invoice.nonreg_credit@2026.1` | `jp_input_tax_credit` | 同上 | [NTA 2026 reform][nta-2026] |
| `jp.invoice.nonreg_credit` | `2030.1` | future | pending | 2026-09-03 | `jp.invoice.nonreg_credit@2028.1` | `jp_input_tax_credit` | 同上 | [NTA 2026 reform][nta-2026] |
| `jp.invoice.nonreg_credit` | `2031.1` | future | pending | 2026-09-03 | `jp.invoice.nonreg_credit@2030.1` | `jp_input_tax_credit` | 経過措置終了後 | [NTA 2026 reform][nta-2026] |
| `jp.invoice.nonreg_cap` | `2026.1` | future | pending | 2026-09-03 | | `jp_input_tax_credit` | 一の相手方からの対象仕入れの年・事業年度合計。課税期間開始日で適用し、超過部分に制限 | [NTA 2026 reform][nta-2026] |
| `jp.records.corp.general` | `unverified` | verify | pending | 2026-09-03 | | `jp_retention_policy` | 確定申告書提出期限翌日から。文書・法人条件と制度適用開始日を確認 | [NTA 5930][nta-5930] |
| `jp.records.corp.loss` | `unverified` | verify | pending | 2026-09-03 | | `jp_retention_policy` | 青色繰越欠損金等の条件と古い事業年度の期間を確認 | [NTA 5930][nta-5930] |
| `jp.payroll.withholding` | `2026.1` | active | pending | 2026-09-03 | | `jp_payroll_calc` | 令和8年分源泉徴収税額表。全明細をdatasetへ取り込む | [NTA 2026 table][nta-withholding] |
| `jp.payroll.health_insurance` | `2026.1` | active | pending | 2026-09-03 | | `jp_payroll_calc` | 協会けんぽ都道府県別率、介護保険等をdatasetへ取り込む。組合健保には別scopeが必要 | [Kyokai Kenpo 2026][kyokai-2026] |
| `jp.payroll.employee_pension` | `unverified` | verify | pending | 2026-09-03 | | `jp_payroll_calc` | 厚生年金の等級、上限、被保険者区分、適用期間を確認後にdatasetを承認する | [Japan Pension Service][nenkin] |
| `jp.payroll.employment_insurance` | `2026.1` | active | pending | 2026-09-03 | | `jp_payroll_calc` | 令和8年度の事業区分別・労使別率をdatasetへ取り込む | [MHLW rates][mhlw-employment] |

## Dataset rules

- dataset entryは率表を取り込んだことを意味しない。公式明細、区分、端数処理、適用期間をd6e SQLのdataset明細表へ投入し、件数と既知例を照合してから`approval=approved`にする。
- 健康保険、厚生年金、雇用保険を一つの`social_insurance`率へまとめない。制度、加入先、都道府県、事業区分、被保険者区分ごとのkeyまたはscopeを保つ。

## Update rules

- 安定keyを維持し、新しい制度値は新versionと新しい半開区間で追加する。
- 令和8年度改正後の経過措置は、古いNo.6498本文だけで判断せず、改正特集と適用対象の課税期間を確認する。
- `verify`行は発見情報であり、適用開始日とscopeを補うまでd6eの承認済みruntime rowにしない。
- d6e側で同じkey、scope、anchorの期間が重複した場合は有効化しない。

[nta-6102]: https://www.nta.go.jp/taxes/shiraberu/taxanswer/shohi/6102.htm
[nta-6498]: https://www.nta.go.jp/taxes/shiraberu/taxanswer/shohi/6498.htm
[nta-2026]: https://www.nta.go.jp/taxes/shiraberu/zeimokubetsu/shohi/keigenzeiritsu/invoice-review/index.htm
[nta-5930]: https://www.nta.go.jp/taxes/shiraberu/taxanswer/hojin/5930.htm
[nta-withholding]: https://www.nta.go.jp/publication/pamph/gensen/zeigakuhyo2026/01.htm
[kyokai-2026]: https://www.kyoukaikenpo.or.jp/about/business/insurance_rate/rate_prefectures/r08/
[nenkin]: https://www.nenkin.go.jp/service/kounen/jigyosho-hiho/jigyonushi/index.html
[mhlw-employment]: https://www.mhlw.go.jp/stf/seisakunitsuite/bunya/0000108634.html
