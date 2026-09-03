# freee連携

Reviewed against the local d6e `d6e-saas-freee` skill on 2026-09-03. API仕様は組み込みskillと接続先の現行仕様を実行時に再確認する。

## 接続境界

- d6e SaaS providerは`freee`を使い、API呼出しには`d6e-saas-freee`を使う。
- 最初に利用可能な事業所を取得し、選択した`company_id`をd6e legal entityへ明示的に対応付ける。
- OAuth credentialはd6e SaaS接続で管理し、ERPのSQLテーブルへ複製しない。
- 一つのcredentialで複数事業所へアクセスできても、sync runを事業所横断で混在させない。

## Object mapping

| freee object | d6e側の用途 | 注意点 |
| --- | --- | --- |
| company | `legal_entities`とのmapping | `company_id`を外部組織IDとして保持する |
| account item | `accounts`とのmapping | 名称一致だけで自動対応しない |
| tax code | `tax_codes`とのmapping | 法定率の正本にせず、`law_params`を参照する分類として扱う |
| partner | `partners`とのmapping | external ID、登録番号、名称変更履歴を分離する |
| deal | source document、open item、journal候補 | detail単位の税区分・金額を検証してから転記する |
| invoice | d6e invoiceとのmapping | dealやpaymentとの関係を確認し、同じ売上を二重転記しない |
| trial balance report | reconciliation evidence | journalの代わりに取り込まない |

内部account codeをfreeeのaccount item IDにしない。mapping rowに適用期間と承認状態を持たせ、名称やIDの変更で過去仕訳の意味を変えない。

## Source-of-truth rules

- d6eが正本なら、d6eで承認済みのinvoiceまたはjournalだけをoutboxへ載せる。
- freeeが既存会計の正本なら、dealとjournal相当データをinboundとして扱い、d6e側は同じsource keyを一度だけ転記する。
- invoice、deal、bank-derived transactionを同時に読む場合は、どのobjectが会計イベントを表すかを決める。すべてを独立journalとして扱わない。
- freee上の更新・削除がposted journalへ影響する場合、差分を上書きせず、d6eでreversalとreplacementを作ってexternal versionを関連付ける。

## Checkpointと再実行

endpointごとに利用可能なupdated-time filter、page、cursorを組み込みskillで確認し、その値をsync runへ保存する。取得順序だけに依存せず、境界時刻を重ねて再取得し、external IDとversionまたはdigestで重複排除する。

## Reconciliation

- 対象期間のdeal、invoice、paymentの件数と金額
- account item・tax code別の金額
- freeeのtrial balanceとd6eのtrial balance
- 未対応account、partner、tax code、取消・削除object
- d6e outbox rowとfreee response IDの一対一対応

差異を自動調整仕訳で消さない。差異原因、source object、mapping version、承認者を記録する。
