# Core accounting model

このモデルは国別制度を載せる土台であり、完成済みの法定勘定科目表ではない。

## Recommended tables

| Table | Purpose | Important keys |
| --- | --- | --- |
| `legal_entities` | 法人・帳簿単位 | `id`, `code`, `base_currency` |
| `currencies` | 通貨と精度 | `code`, `minor_units` |
| `fiscal_periods` | 会計期間と締め | `entity_id`, `start_date`, `end_date`, `status` |
| `accounts` | 勘定科目 | `entity_id`, `code`, `account_type`, `normal_side` |
| `partners` | 顧客・仕入先等 | `entity_id`, `code`, tax registration fields |
| `tax_codes` | 取引の税分類master | `jurisdiction`, `code`, classification, `law_param_id` |
| `journals` | 仕訳ヘッダー | `entity_id`, `journal_no`, `status`, source identity |
| `journal_lines` | 借方・貸方明細 | `journal_id`, `line_no`, `account_id`, `side`, `amount` |
| `doc_links` | 原始証憑との関係 | `document_type`, `document_id`, `storage_file_id` |

すべて23文字以内である。物理名を増やす前に、既存テーブルとの重複を調べる。

法定rateと控除割合のsystem of recordは`law_params`とし、`tax_codes`へ独立した正本を複製しない。計算結果へrateをsnapshotする場合は、使用した`law_params`行のIDとversionも保存する。

## Header and line rules

`journals`には少なくとも次を持たせる。

- 法人、仕訳番号、仕訳日、会計期間
- `draft`、`posted`、`reversed`などの状態
- source system、source type、source key
- 取引通貨と機能通貨
- 転記者、転記時刻、取消元仕訳
- 説明、作成・更新時刻

`journal_lines`には少なくとも次を持たせる。

- 仕訳、行番号、勘定科目
- `debit`または`credit`
- 正の取引通貨額
- 機能通貨額と使用した為替レート
- 税区分、取引先、部門などの分析軸

明細の金額へ正負を混在させない。金額は正数、向きは`side`で表す。報告クエリでは、借方を正、貸方を負など一つの符号規約へ変換する。

保存列には桁数を定めた`NUMERIC`を使う。SQLからSTFへ読むときは金額と為替レートを`::text`で返し、JavaScriptの浮動小数点へ変換しない。通貨の最小単位が固定される範囲では整数最小単位も選べるが、通貨ごとの桁数と丸め規則をマスタで明示する。

## Constraints

可能な範囲をSQL制約で守る。

- 法人内の勘定科目コード、仕訳番号を`UNIQUE`にする。
- `journal_lines(journal_id, line_no)`を`UNIQUE`にする。
- 状態、勘定種別、通常残高、明細sideを`CHECK`で限定する。
- `amount > 0`、`fx_rate > 0`を`CHECK`する。
- source systemとsource keyの組を一意にする。
- 外部キーで法人、期間、勘定、税区分、仕訳を結ぶ。

仕訳全体の借貸一致や締め期間判定は行をまたぐため、転記STFで検証する。アプリケーションだけに任せず、転記経路を一つに限定する。

## State transitions

```text
draft -> posted -> reversed
```

- `draft`: 編集可能。元帳残高へ含めない。
- `posted`: 編集不可。元帳残高へ含める。
- `reversed`: 元仕訳自体は保持し、反対仕訳への参照を持つ。

取消後の正しい内容は別の新規仕訳として転記する。過去の事実を上書きしない。

## Reporting

試算表は転記済み明細だけを対象に、勘定科目ごとの借方・貸方・差額を集計する。損益と財政状態の分類は勘定科目マスタの`account_type`から導く。国別表示科目へのマッピングは国別スキルに置く。

決算検証では少なくとも次を突合する。

- 全転記仕訳の借方合計 = 貸方合計
- 補助元帳合計 = 総勘定元帳統制勘定
- 売掛・買掛残高 = 未消込明細合計
- 税統制勘定 = 税区分別集計
- 期首残高 + 当期増減 = 期末残高
