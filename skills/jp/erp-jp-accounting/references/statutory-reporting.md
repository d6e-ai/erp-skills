# 日本の法定・税務報告

Official information reviewed: 2026-09-03. Forms and electronic filing specifications change; verify the applicable fiscal year before generating output.

## Reporting layers

次の層を分離する。

1. d6e SQL上の元帳・補助元帳
2. 企業の財務諸表表示科目
3. 税務申告、内訳明細、電子申告の項目

一つの勘定科目名をすべての層で使い回さず、バージョン付きマッピングを持つ。

## Required traceability

生成した報告データに次を記録する。

- 法人、会計年度、対象期間
- 元帳の締め状態と抽出時刻
- 使用した科目マッピングと様式バージョン
- 生成者、承認者、生成ファイルID
- 合計値と元帳へのドリルダウンキー

## Validation

- 貸借が一致する。
- 貸借対照表の期首・当期増減・期末が連続する。
- 損益計算書の当期損益と純資産増減の関係を説明できる。
- 勘定科目内訳と総勘定元帳統制勘定が一致する。
- 申告出力を再生成して同一条件で同一結果になる。

Official starting points:

- [国税庁 法人税の申告・手続](https://www.nta.go.jp/taxes/tetsuzuki/shinsei/annai/hojin/mokuji.htm)
- [e-Tax 法人税申告](https://www.e-tax.nta.go.jp/hojin/gimuka/index.htm)

d6eから直接提出できると仮定しない。まず検証可能な中間データまたはファイルを生成し、対象年度の提出仕様と外部接続を別途確認する。
