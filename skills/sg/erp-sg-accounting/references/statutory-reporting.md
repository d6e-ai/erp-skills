# Singapore statutory reporting

Official information reviewed: 2026-09-03. Company classification, exemptions, filing rules, and taxonomies change; verify them for the filing date.

## Separate reporting layers

1. d6e SQL上のledgerとsubledgers
2. entityのfinancial statements
3. ACRA XBRL taxonomy mapping
4. IRAS GSTおよびcorporate income tax reporting data

一つのaccount codeをACRA/IRAS fieldへ直接固定せず、report type、version、effective dates付きmapping tableを持つ。

## ACRA XBRL

ACRAは会社分類によりfinancial statementsの提出要否とFull XBRL、Simplified XBRL、specialised templates、PDF等を区別している。対象会社の分類を確認せずtemplateを選ばない。

適用するACRA taxonomyと開始日は[parameter-inventory.md](parameter-inventory.md)を参照し、taxonomy versionをコードへ固定しない。生成時に対象versionとvalidation rulesを記録する。

Official sources:

- [ACRA: Filing financial statements in XBRL format](https://www.acra.gov.sg/manage/companies/legal-requirements-common-offences/filing-financial-statements-in-xbrl-format/)
- [ACRA: Filing requirements and exemptions](https://www.acra.gov.sg/manage/companies/legal-requirements-common-offences/filing-financial-statements-in-xbrl-format/requirements-exemptions/)

## Output traceability

- entity、UEN、financial year end、reporting period
- ledger close IDとextract timestamp
- accounting framework、mapping version、taxonomy version
- generated file ID、validation result、preparer、approver
- report lineからaccount/journalへ戻るdrill-down key

## Validation

- trial balanceのdebitとcreditが一致する。
- subledgerとcontrol accountが一致する。
- financial statementsのcomparativesとopening balancesが連続する。
- XBRL validation rulesを対象taxonomy versionで通す。
- GST return集計からinvoice、credit note、journalへ遡れる。

d6eがACRAやIRASへ直接提出できると仮定しない。まず検証可能な中間データまたはファイルを生成し、提出接続は`erp-sg-integrations`で別途扱う。
