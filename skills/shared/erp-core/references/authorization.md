# ERP authorization on d6e

## Roles are duties, not screens

権限は画面単位ではなく、実行できる会計職務として設計する。典型例は次のとおり。

| Duty | Typical access |
| --- | --- |
| Master data maintainer | マスタの参照・登録・限定更新 |
| Preparer | draft伝票の作成・自分のdraftの更新 |
| Approver | draftの参照・承認、自己作成分の承認は不可 |
| Poster STF | 承認済み伝票の転記に必要な表だけを更新 |
| Accountant | 元帳、照合、期間締めの実行 |
| Auditor | 全期間の読取専用、監査ログ参照 |
| Integration STF | 指定source systemの取込表と必要な伝票作成だけ |

組織規模に応じて職務をまとめてもよいが、作成・承認・転記の主体を監査できる状態は保つ。

## d6e Policy mapping

1. ユーザーまたはSTFをPolicy Groupへ追加する。
2. 表ごと、`select`、`insert`、`update`、`delete`ごとにPolicyを定義する。
3. 条件付きPolicyでは法人ID、状態、担当者などを使い、他法人や転記済み行を対象外にする。
4. 書込みを行う保存済みSTFには、そのSTF専用Policy Groupを作る。
5. DDL権限は通常運用から分離し、`ddl_policy_group`を最小限にする。

PolicyがないDMLは拒否される。管理者で確認できたことを一般ユーザーやSTFの成功とみなさない。

条件はSQL文字列ではなくJSONオブジェクトで渡す。確認した現行API実装が扱う比較演算子は`$eq`、`$ne`、`$gt`、`$gte`、`$lt`、`$lte`、`$in`、`$null`で、現在の主体IDは文字列`"$user_id"`として参照する。ツール説明との相違がある場合は、対象d6e版のAPI実装と実行結果を優先する。

## Current enforcement limits

2026-09-03に確認した現行実装には、会計設計上無視できない制約がある。

- 条件付き`insert` Policyは所属による許可判定には使われるが、条件式が挿入行へ適用されない。法人ID、状態、source systemなどの挿入条件は保存済みSTFで検証し、一般ユーザーへ対象表の直接`insert`を許可しない。
- JOINを含むDMLでは、PolicyのWHERE句は解析された先頭テーブルを基準に一つだけ構築される。機密表同士をJOINして認可境界を作らず、主体ごとに許可された単表取得または専用STFへ分ける。
- 複雑なCTE、集合演算、サブクエリを認可境界として使わない。許可・拒否の両方を実行してから採用する。

これらはプラットフォーム側で改善され得る。対象バージョンで挙動が変わった場合は、検証結果とd6eの現行仕様を優先する。

## Accounting protections

- 転記済み仕訳へのユーザー直接`update`と`delete`を許可しない。
- 転記用STFだけが`journals.status = 'posted'`への遷移と明細確定を行う。
- 期間締めと再オープンを別権限にする。
- 自己承認を禁止する場合、preparerとapproverのIDをSTFで比較する。
- Integration STFは自身のsource system以外のsource keyを作成できないよう検証する。
- 監査役には会計表と監査ログの読取だけを与える。

## Verification matrix

各主体について成功ケースだけでなく拒否ケースも実行する。

| Subject | Allowed test | Denied test |
| --- | --- | --- |
| Preparer | draft作成 | posted仕訳更新 |
| Approver | 他者draft承認 | 自己承認 |
| Poster STF | 承認済み転記 | 未承認・締め期間転記 |
| Auditor | 全件参照 | insert/update/delete |
| Integration STF | 指定sourceの取込 | 他sourceの偽装・重複取込 |
