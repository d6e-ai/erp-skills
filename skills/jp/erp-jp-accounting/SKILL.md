---
name: erp-jp-accounting
description: >-
  Design and implement Japanese ERP accounting on d6e SQL, including Japanese charts of accounts, consumption tax, qualified invoices, payroll accounting, statutory reporting, and record retention. Use for Japan-specific accounting requirements; use erp-jp-integrations for external service synchronization.
---

# 日本向けd6e基幹会計

d6e SQLをsystem of recordとして、日本法人・日本拠点向けの会計モデル、税区分、請求、給与仕訳、法定報告、保存要件を設計・構築する。

## Required platform skills

- 共通の会計モデルと不変条件には`erp-core`を使う。
- DDL/DMLには組み込み`d6e-sql`、業務更新には`d6e-stf`、複数ステップには`d6e-workflow`、認可には`d6e-policy`を使う。
- d6e外の独自DBや旧Trebit台帳を作らない。

## Confirm before implementation

次の事業者条件で設計が変わる。ワークスペースの既存データから確認できない項目だけを質問する。

- 法人・個人、会計年度、機能通貨
- 消費税の課税・免税区分、課税方式、経理方式
- 適格請求書発行事業者の登録状態と有効期間
- 給与計算をd6e内で行うか、外部給与サービスの結果だけを仕訳するか
- 外部会計サービス、銀行、請求サービスのsource of truth
- 必要な帳票・申告データと保存期間

税務判断を推測しない。制度、税率、経過措置、帳票仕様は実装時点の公式情報を再確認し、企業固有の判断は税理士等の確認対象として明示する。

## References

- 勘定科目と日本向け表示: [references/chart-of-accounts.md](references/chart-of-accounts.md)
- 消費税区分、税率、仕入税額控除: [references/consumption-tax.md](references/consumption-tax.md)
- 適格請求書と請求データ: [references/invoicing.md](references/invoicing.md)
- 給与、源泉徴収、社会保険の会計連携: [references/payroll.md](references/payroll.md)
- 税率、控除割合、期限、率表・様式バージョン: [references/parameter-inventory.md](references/parameter-inventory.md)
- 法人税申告添付・財務報告への出力: [references/statutory-reporting.md](references/statutory-reporting.md)
- 帳簿書類と電子取引の保存: [references/record-retention.md](references/record-retention.md)

依頼に関係する参照だけを読む。例えば請求書作成では、給与参照を読み込まない。

## d6e implementation rules

- 事業者、会計期間、勘定科目、取引先、税区分、仕訳、証憑参照をd6e SQLテーブルで保持する。
- 税率、控除割合、給与税額表、社会保険率、様式バージョンをSTFへハードコードしない。`parameter-inventory.md`をseedとして確認し、実行時は安定key、適用基準日、適用期間、承認状態を持つd6e SQLマスタから取得する。
- 適格請求書登録番号は取引先の単一文字列で終わらせず、登録状態と有効期間の履歴を保持できるようにする。
- 税抜・税込、税率区分、課税区分、税額、端数処理方法を、原始証憑から仕訳明細まで追跡できるようにする。
- 転記、取消、締め、税集計は保存済みSTFを唯一の更新経路とし、Policyで直接更新を制限する。
- 法定出力は元帳データを上書きせず、対象期間、生成時刻、ロジックまたは様式バージョンを記録する。

## Verification

既知の小さな数値例で、売上、仕入、返品、免税・非課税・不課税、複数税率、端数、締め、取消を検証する。実行済みのd6e結果、外部サービス未接続、専門家確認待ちを分けて報告する。
