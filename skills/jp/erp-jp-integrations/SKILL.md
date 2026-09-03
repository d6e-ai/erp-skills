---
name: erp-jp-integrations
description: >-
  d6e SQLを正本とする日本向けERPをfreee、マネーフォワード クラウド、銀行明細などの外部サービスと安全に同期する。日本向けの会計・請求・経費・銀行連携、マッピング、再実行、照合を設計・実装するときに使う。制度判定と仕訳規則にはerp-jp-accountingを使う。
---

# 日本向けERP外部連携

d6e SQLをsystem of recordとして維持しながら、日本の会計SaaS、請求・経費サービス、銀行データと連携する。

## 必須スキルと境界

- 共通元帳、仕訳、認可には`erp-core`を使い、日本固有の税務・請求・給与要件には`erp-jp-accounting`を使う。
- freeeには組み込み`d6e-saas-freee`、マネーフォワード クラウド会計・請求には`d6e-saas-moneyforward-cloud`、経費には`d6e-saas-moneyforward-expense`を使う。APIのendpointやpayloadをこのスキルから推測しない。
- 組み込みSaaS skillがない接続先には`d6e-effect`と`d6e-workflow`を使う。credentialやtokenをERPテーブル、STFコード、ログ、証憑へ保存しない。
- 外部サービスの接続成功を会計転記の成功とみなさない。外部同期状態とd6eの仕訳状態を独立して管理する。

## 実装前に確定すること

- 接続先、接続先の法人・事業所ID、対応するd6e legal entity
- partner、account、tax classification、invoice、expense、journal、paymentごとのsource of truth
- inbound、outbound、双方向のどれかと、競合時の優先順位
- 初回取込期間、増分取得条件、締め済み期間の扱い
- 外部writeを許可する操作、承認者、dry-runまたはsandboxの有無
- 外部側で削除・取消・再採番されたデータの訂正方法

同じ業務データを複数サービスから取り込む場合、二重仕訳を防ぐためcanonical sourceとcross-service keyを先に決める。

## d6e SQL同期モデル

接続情報、object mapping、実行履歴、item単位の結果、outbox、webhook inboxをd6e SQLに保持する。テーブル名は23文字以下にし、credential本体は含めない。

最低限、次を追跡する。

- provider、connection key、entity ID、external organization ID
- object type、external ID、internal table、internal ID、external versionまたはdigest
- sync direction、開始・終了時刻、checkpoint、件数、status
- itemごとのsource key、payload digest、処理結果、error category、retry count
- outbound idempotency key、request version、response ID、acknowledged at
- webhook event ID、received at、payload digest、processing status

外部IDの一意性はprovider、connection、object typeを含む範囲で保証する。外部IDだけを全接続共通の識別子にしない。

SQL constraintで、webhookはprovider・connection・event ID、outboxはprovider・connection・idempotency key、sync itemはrun・source key・source versionまたはdigestの重複を拒否する。STFでpending rowを条件付きUPDATEしてclaimまたはleaseを取得し、同じinbox/outbox itemを複数Workflowが同時実行できないようにする。

## 同期フロー

### Inbound

1. 組み込みSaaS skillまたはEffectで、明示した期間・cursor・pageだけを取得する。可能ならsnapshot cursorまたは固定した上限watermarkと安定した`(updated_at, external_id)`順序を使う。
2. raw responseのdigest、取得時刻、endpoint識別子をsync itemへ記録する。
3. STFでmapping、型、通貨、税区分、重複、締め期間を検証する。
4. posted journalが必要なら、外部取込専用のposting STFを通す。API responseから直接`journal_lines`へINSERTしない。
5. すべてのpageとitemをdurableなsync itemとして記録してからfetch checkpointを進める。fetch checkpointと処理完了を混同せず、失敗itemを再試行可能な状態で残し、未解決のままrunを完了扱いにしない。

offset paginationしかなく取得中に並び順が変わり得るAPIでは、重複排除だけで欠落を防げない。境界を重ねた再scanと期間・件数・金額のreconciliationを必須にし、欠落可能性を検証済みと報告しない。

### Outbound

1. STFで送信対象を確定し、同じtransaction内でoutbox rowと不変のidempotency keyを作る。
2. WorkflowとSaaS proxyまたはEffectで外部へ送信する。
3. 別のSTFでresponse ID、外部version、成功・失敗を記録する。
4. retryでは同じoutbox rowとidempotency keyを再利用する。元invoiceやjournalを再生成しない。

endpointごとにproviderがidempotency keyを受理・保証するか記録し、保証される場合だけ自動再送する。保証されないendpointでtimeoutなど結果不明になったattemptは`indeterminate`にし、deterministicな外部referenceでread-backまたはreconciliationを行い、未解決のまま再送しない。

外部write、取消、削除はユーザーの依頼範囲と承認を確認してから実行する。posted journalに対応する外部変更は、物理削除ではなく取消・反対仕訳・replacement mappingとして扱う。

## References

- freeeのobject mappingと照合: [references/freee.md](references/freee.md)
- マネーフォワード クラウド会計・請求・経費の境界: [references/moneyforward.md](references/moneyforward.md)
- 銀行明細の取込、matching、reconciliation: [references/banking.md](references/banking.md)

依頼に関係するreferenceだけを読む。

## 検証

- 同じpage、webhook、outboxを再実行しても重複objectや重複仕訳が生じない。
- account、partner、tax codeの未対応mappingを安全に停止できる。
- external totalとd6e totalを期間・通貨・税区分ごとに照合できる。
- rate limit、部分失敗、timeout、遅延responseから再開できる。
- 外部接続未検証、read検証済み、write検証済み、会計照合済みを区別して報告する。
