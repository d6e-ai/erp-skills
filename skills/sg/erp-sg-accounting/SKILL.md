---
name: erp-sg-accounting
description: >-
  Design and implement Singapore ERP accounting on d6e SQL, including Singapore charts of accounts, GST, tax invoices, payroll accounting, ACRA and IRAS reporting, and record retention. Use for Singapore-specific accounting requirements; use erp-sg-integrations for external service synchronization.
---

# Singapore d6e ERP accounting

d6e SQLをsystem of recordとして、シンガポール法人・拠点向けの会計モデル、GST、請求、給与仕訳、ACRA/IRAS報告、保存要件を設計・構築する。

## Required platform skills

- 共通会計モデルと不変条件には`erp-core`を使う。
- DDL/DMLには組み込み`d6e-sql`、業務更新には`d6e-stf`、複数ステップには`d6e-workflow`、認可には`d6e-policy`を使う。
- d6e外の独自DBや旧Trebit台帳を作らない。

## Confirm before implementation

次の条件を既存ワークスペースから確認し、不明な項目だけを質問する。

- 法人種別、UEN、financial year end、機能通貨
- GST登録状態、登録番号、適用開始日、GST accounting periods
- InvoiceNow Requirementの対象時期と、利用するsolution providerまたはAccess Point
- 給与計算をd6e内で行うか、外部給与サービスの結果だけを仕訳するか
- Singapore Citizen、Permanent Resident、外国人従業員を扱う範囲
- ACRAへのfinancial statements/XBRL提出要否と会社分類
- 外部会計、銀行、給与、請求サービスのsource of truth

税務・雇用・提出要件を推測しない。制度、率、threshold、段階導入日、taxonomyは公式情報を実装時に再確認し、専門家確認が必要な企業固有判断を明示する。

## References

- 勘定科目とSingapore reporting mapping: [references/chart-of-accounts.md](references/chart-of-accounts.md)
- GST区分、税率、input/output tax: [references/gst.md](references/gst.md)
- Tax invoiceとInvoiceNow-ready data: [references/invoicing.md](references/invoicing.md)
- CPF、payroll accounting、itemised payslips: [references/payroll.md](references/payroll.md)
- GST rate、threshold、deadline、CPF/XBRL version: [references/parameter-inventory.md](references/parameter-inventory.md)
- ACRA XBRLとIRAS reporting: [references/statutory-reporting.md](references/statutory-reporting.md)
- Accounting/GST recordsの保存: [references/record-retention.md](references/record-retention.md)

依頼に関係する参照だけを読む。

## d6e implementation rules

- entity、period、account、partner、tax code、journal、invoice、payment、evidence linkをd6e SQLテーブルで保持する。
- GST rate、CPF rate、wage ceiling、filing threshold、InvoiceNow phase、ACRA taxonomyをSTFへハードコードしない。`parameter-inventory.md`をseedとして確認し、実行時は安定key、適用基準日、適用期間、承認状態、根拠URLを持つd6e SQLマスタから読む。
- UEN、GST registration number、Peppol ID、外部system IDを用途別に保持し、一つの識別子へ混同しない。
- foreign currency invoiceでは取引通貨額、GST purposeのSGD換算額、使用rate、rate source、rate dateを追跡する。
- 転記、取消、close、GST aggregationは保存済みSTFを唯一の更新経路とし、Policyで直接更新を制限する。
- InvoiceNow送信は元帳転記と疎結合にし、送信失敗によって同じ仕訳を二重作成しない。
- statutory outputは対象期間、抽出時刻、mapping/taxonomy version、生成ファイルIDを記録する。

## Verification

小さな数値例でstandard-rated、zero-rated、exempt、out-of-scope、credit note、foreign currency、GST rounding、CPF rounding、close/reversalを検証する。実行済みd6e結果、外部接続未検証、専門家確認待ちを区別して報告する。
