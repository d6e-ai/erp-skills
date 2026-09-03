# マネーフォワード クラウド連携

Reviewed against the local d6e MoneyForward SaaS skills on 2026-09-03. endpoint、scope、利用条件は実行時に組み込みskillと接続先の現行仕様を確認する。

## 組み込みskill

- クラウド会計・請求: `d6e-saas-moneyforward-cloud`、provider=`moneyforward_cloud`
- クラウド経費: `d6e-saas-moneyforward-expense`、provider=`moneyforward_expense`
- SaaS proxyでは会計pathに`/accounting`、請求pathに`/invoice`のrouting prefixが必要。具体的なpathとpayloadは組み込みskillを正本とする。

OAuth tokenが表す事業者・officeを取得し、選択したd6e legal entityへ明示的にbindingする。サービス間でoffice IDやobject IDが同じ意味だと仮定しない。

## Object boundaries

| Service object | d6e側の用途 | 注意点 |
| --- | --- | --- |
| office / accounting period | entityとperiodの照合 | d6eの締め状態を外部値で暗黙更新しない |
| account / sub-account | account・dimension mapping | 外部IDを内部account codeにしない |
| tax | `tax_codes`とのmapping | 法定率はd6e `law_params`を正本とする |
| trade partner | `partners`とのmapping | 法人番号とinvoice登録番号を別項目で保持する |
| journal / branch | journal header・debit/credit lines | external journal IDとbranchの順序を保存する |
| connected-account transaction | bank statement candidate | posted journalと同一視しない |
| billing / quote | invoice・sales document | accounting journalとのlinkを追跡する |
| voucher | evidence link | file ID、digest、journalとの関係を保存する |
| expense transaction / report | expense・approval source | 会計側journalとcross-service deduplicationする |

## 同期判断

- クラウド会計journalをinboundの会計正本にする場合、請求・経費から同じ取引を再度転記しない。
- d6e journalをoutbound送信する場合、debit/credit、tax、department、sub-account、partner mappingがすべて承認済みでなければ送信しない。
- connected-account transactionは照合候補であり、外部側でjournal化済みか確認するまでd6e journalを自動生成しない。
- billingの発行状態、入金状態、会計転記状態を一つのstatusへ畳み込まない。
- voucherの保存成功をjournal作成成功の代用にしない。

## 再実行と訂正

各provider、service、office、object type、external IDを含むsource keyを使う。更新ではexternal update timeまたはpayload digestを比較し、同一versionを再処理しない。posted journalの変更は上書きせず、reversal、replacement journal、external mappingを一組で作る。

## Reconciliation

- journal件数、debit・credit合計、tax区分別合計
- account、sub-account、department、partnerの未対応mapping
- billing、payment、journalのlink不足
- expense reportと会計journalの重複・欠落
- voucherのdigestと取得可能性
- external trial balanceとd6e trial balance

照合対象のservice、office、期間、取得時刻、API filter、mapping versionを結果へ記録する。
