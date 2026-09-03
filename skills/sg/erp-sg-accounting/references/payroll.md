# Singapore payroll accounting

Official information reviewed: 2026-09-03. CPF rate-table versions are listed in [parameter-inventory.md](parameter-inventory.md); rates, ceilings, and employment requirements must be re-checked for the applicable pay date and employee.

給与計算と、外部payroll systemで確定した結果の会計転記を分ける。転記だけが要件なら、d6e内にCPF計算を複製しない。

Official sources:

- [CPF Board: How much CPF contributions to pay](https://www.cpf.gov.sg/employer/employer-obligations/how-much-cpf-contributions-to-pay)
- [CPF contribution calculator](https://www.cpf.gov.sg/employer/tools-and-services/calculators/cpf-contribution-calculator)
- [MOM: Itemised pay slips](https://www.mom.gov.sg/employment-practices/salary/itemised-payslips)
- [MOM: Salary](https://www.mom.gov.sg/employment-practices/salary)

## Effective-dated inputs

CPF computation depends on more than one percentage. At minimum, retain or import:

- pay date and salary period
- citizenship/PR status and SPR year
- age band effective for the contribution month
- Ordinary Wages and Additional Wages
- applicable wage ceilings and contribution table version
- employer share、employee share、rounding results

特定年度の値をSTFに固定しない。公式tableまたはcalculatorと照合できるeffective-dated SQL masterを使う。

## Payroll result model

- payroll run ID、employee ID、period、payment date
- basic salary、allowances、bonus、overtime、other earnings
- employee CPF、other deductions、net salary
- employer CPF and other employer costs
- rate table version、calculation source、approval status
- payment ID、journal ID、reversal run ID

給与明細に必要な個人情報と、総勘定元帳に必要な集計を分離し、Policy Groupを分ける。

## Posting example

Gross salary S$5,000、employee deductions S$1,000、net salary S$4,000なら、概念上はdebit employee cost 5,000、credit deductions payable 1,000、credit salary payableまたはbank 4,000となる。Employer CPF等は別のdebit employee costとcredit payableとして計上する。

この例は科目構造の説明であり、CPF eligibilityや率を確定するものではない。

## Controls

- payroll runを一意にして二重仕訳を拒否する。
- CPFのtotal、employee share、employer shareとroundingを個別に保持する。
- calculation、approval、payment、postingの主体を追跡する。
- correction runを元runと元journalへ関連付ける。
