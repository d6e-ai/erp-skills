# 日本の銀行データ連携

銀行API、aggregation service、CSV、全銀形式など、実際に利用できる取得経路を先に確認する。未確認のendpoint、認証方式、field layoutを推測しない。

## Source boundary

- credential、同意情報、refresh tokenをERPテーブルへ保存しない。
- d6e SaaS skillがあるproviderはそのskillを使い、ない場合は認証済みEffectまたは承認されたfile importを使う。
- screen scrapingや個人向けlogin credentialの保存を暗黙に採用しない。
- bank account、virtual account、credit card、payment processorを別connectionとして識別する。

## SQL model

`bank_stmts`と`bank_lines`または同等の短いテーブルに、最低限次を保持する。

- entity ID、internal bank account ID、provider、external account ID
- statement ID、period、opening balance、closing balance、currency
- provider transaction ID、transaction date、value date、amount、direction
- description、counterparty、reference、raw payload digest
- source file IDまたはsync run ID、imported at、mapping version
- match status、matched payment ID、matched journal ID、reviewer

金額は浮動小数点にせず、符号の意味をprovider単位で明示して正規化する。provider transaction IDが安定しない場合は、account、date、amount、reference、raw digestを組み合わせたdeduplication keyも保持する。

## Matching

1. 既存のpayment、open item、journalとの明示的reference一致を優先する。
2. amount、currency、date、counterpartyによる候補を作る。
3. 一意でない候補や分割・合算入金はreview待ちにする。
4. 承認済みmatchだけをsettlement STFへ渡す。

fuzzy matchを会計事実として自動確定しない。bank line自体をjournalとして扱わず、未処理、候補、照合済み、除外の状態を分ける。

## Reconciliation

- statementのopening balance + movements = closing balanceを確認する。
- statement period内の全lineが、matched、approved exclusion、unresolvedのいずれかになるようにする。
- bank control accountとreconciled closing balanceの差異を表示する。
- correction、duplicate、reversal lineを元lineへ関連付ける。
- file importの再実行とAPI再取得で同じlineが増えないことを確認する。

差異を自動のplug entryで相殺しない。差異が残る場合は金額、通貨、line ID、候補、理由を報告する。
