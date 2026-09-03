# 日本の帳簿・証憑保存

Official information reviewed: 2026-09-03. Retention periods and electronic record requirements depend on entity, period, document, and special circumstances.

保存年数と例外は[parameter-inventory.md](parameter-inventory.md)を参照する。起算点と例外条件があるため、単一の固定期間を全レコードへ適用しない。

Official sources:

- [国税庁 No.5930 帳簿書類等の保存期間](https://www.nta.go.jp/taxes/shiraberu/taxanswer/hojin/5930.htm)
- [国税庁 電子取引関係](https://www.nta.go.jp/law/joho-zeikaishaku/sonota/jirei/tokusetsu/01.htm)

## SQL and storage design

証憑参照または保存方針に次を持たせる。

- 文書種別、法人、取引日、会計年度
- 発行者・受領者、取引金額、税区分
- d6e storage file ID、元ファイル名、MIME type
- 取得元、取得日時、ファイルdigest
- 関連する請求、支払、仕訳ID
- retention policy ID、保存期限、legal hold
- 訂正・差替え前後の関係

`deleted_at`だけで保存義務を満たすと仮定しない。保存期限前の物理削除をPolicyと業務STFで拒否し、原本性、検索性、改ざん防止など対象制度の要件を実装時点の公式Q&Aで確認する。

電子取引データを印刷物だけに変換して完了としない。受領した電子データと取引情報を追跡できるようにする。
