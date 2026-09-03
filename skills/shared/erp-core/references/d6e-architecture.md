# d6e implementation boundary

Verified against the local d6e source on 2026-09-03.

## SQL execution

- `d6e_sql`はワークスペース内の非修飾テーブル名を`user_data.ws_{workspace_id}_...`へ変換する。接頭辞を手書きしない。
- ワークスペース接頭辞が40文字、PostgreSQL識別子上限が63文字のため、ユーザーテーブル名は23文字以内にする。名前は小文字で始め、小文字英数字と単一のアンダースコアだけを使う。`pg_`始まり、末尾アンダースコア、連続アンダースコア、SQL予約語は使えない。
- 一回の`d6e_sql`呼び出しで許可されるSQL文は一文だけである。
- 現行実装が扱うのは`CREATE TABLE`、対応範囲内の`ALTER TABLE`、`DROP TABLE`、`SELECT`、`INSERT`、`UPDATE`、`DELETE`である。
- 現行実装は`CREATE INDEX`を受理しない。索引が必要な規模では、成功したと仮定せずd6e側の機能追加または現行バージョンの対応状況を確認する。
- 外部キーの`ON DELETE`と`ON UPDATE`は`RESTRICT`または`NO ACTION`だけを使える。`CASCADE`、`SET NULL`、`SET DEFAULT`を前提にしない。
- DDLはワークスペース管理者または`ddl_policy_group`のメンバーが実行する。STF内の`sql()`からDDLは実行できない。
- DMLはPolicyの既定拒否を受ける。表を作っただけでは利用できない。
- CTE、`UNION`などの集合演算、式内サブクエリは、すべてのテーブル名がワークスペース名へ変換されると仮定しない。会計処理では単純な一文を優先し、必要な場合は対象d6e版で`executed_sql`と拒否ケースを先に検証する。

## Columns

d6eの一部ドキュメントにはシステム列を自動追加する記述があるが、確認した現行SQL変換コードは列を注入しない。次のような必要列はDDLで明示する。

```sql
id UUID PRIMARY KEY DEFAULT uuidv7(),
created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
deleted_at TIMESTAMPTZ
```

対象インスタンスの`executed_sql`と実テーブルを確認し、挙動が変わっている場合は実行結果を優先する。

`updated_at`を自動更新するトリガーも前提にしない。更新SQLで`updated_at = now()`を明示する。

## STF runtime

- STFコードは関数宣言で包まず、トップレベルに書き、最後に`return`する。
- `$input`は入力、`$caller`は認証済みユーザーIDまたは`null`、`$sources`はWorkflowの入力ステップである。
- `sql(query)`は引数を一つだけ取る。`SELECT`は行配列、DMLは影響行数を返す。
- 一つのSTF内のSQL呼び出しは一つのDBトランザクションを共有する。転記の検証と全明細の書込みを同じSTFに置く。
- 保存済みSTFのSQL権限はSTF主体に対するPolicyで与える。即時実行コードは呼出ユーザー主体のPolicyを受ける。
- 入力値をSQL文字列へ直接補間しない。UUID、日付、列挙値、金額を形式検証し、文字列リテラルはPostgreSQL規則でエスケープする。識別子は固定の許可リストからだけ選ぶ。
- `NUMERIC`はSQL内では正確でも、`SELECT`結果はJSONを経由する。STFへ金額を返すときは`amount::text`のように文字列化し、JavaScriptの`Number`へ変換しない。

## Workflow runtime

- 単一のQJS STFで完結する処理をWorkflowで包まない。
- Workflowの各STFは個別トランザクションで動く。複数STFにまたがる原子的コミットを仮定しない。
- 監査上再現性が必要なWorkflowではSTF/Effectのバージョンを固定する。それ以外は最新版追従の影響を明示する。
- field mappingの変数パスは`$input`、`$sources`、`$steps[n]`から始める。

## Verification

作成を報告する前に、実際のツール結果で次を確認する。

1. テーブルと制約が存在する。
2. 対象主体が許可された操作を実行できる。
3. 対象外主体が拒否される。
4. 正常仕訳が転記できる。
5. 不均衡、締め期間、重複source keyが拒否される。
6. 残高と帳票集計が既知の数値例に一致する。
