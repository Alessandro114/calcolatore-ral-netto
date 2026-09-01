# RAL → Net Salary Calculator

A working prototype of a calculator that, given a gross annual salary (RAL — *Retribuzione Annua Lorda*), returns the employee's annual and monthly net pay, with a breakdown of every withholding.

**[→ Live demo on GitHub Pages](https://alessandro114.github.io/calcolatore-ral-netto/)**

---

## Contents

- [How it works](#how-it-works)
- [Calculation pipeline](#calculation-pipeline)
- [Breakdown of each component](#breakdown-of-each-component)
  - [1. Employee INPS contributions](#1-employee-inps-contributions)
  - [2. Fringe benefits](#2-fringe-benefits)
  - [3. Supplementary pension fund](#3-supplementary-pension-fund)
  - [4. IRPEF 2025](#4-irpef-2025)
  - [5. Employee tax deductions](#5-employee-tax-deductions)
  - [6. Dependent family deductions](#6-dependent-family-deductions)
  - [7. Tax wedge bonus 2025](#7-tax-wedge-bonus-2025)
  - [8. Supplementary treatment](#8-supplementary-treatment)
  - [9. Regional IRPEF surtax](#9-regional-irpef-surtax)
  - [10. Municipal IRPEF surtax](#10-municipal-irpef-surtax)
  - [11. TFR — Severance pay](#11-tfr--severance-pay)
  - [12. Employer cost](#12-employer-cost)
- [Available inputs](#available-inputs)
- [Generated outputs](#generated-outputs)
- [Simplifications and known limits](#simplifications-and-known-limits)
- [Project structure](#project-structure)
- [How to run](#how-to-run)

---

## How it works

The calculator is a **single HTML page** with no external dependencies (zero frameworks, zero build step). All the tax logic is implemented in vanilla JavaScript with isolated, documented functions.

The user enters the parameters → clicks "Calculate" → gets:
- **Summary cards** with annual net, monthly net, withholdings and employer cost
- **Visual bar** showing the composition of the RAL (net vs. taxes)
- **Breakdown table** with every line item of the calculation
- **IRPEF detail** by bracket
- **Regional surtax detail** by bracket
- **TFR section** with accrual and estimated separate taxation
- **Tax wedge detail** (when applicable)
- **List of assumptions** applied

---

## Calculation pipeline

```
RAL (Gross Annual Salary)
  + Taxable fringe benefits (if they exceed the exemption threshold)
  − Pension fund contribution (deductible, max €5.164,57)
  − Employee INPS contributions
  ─────────────────────────────────────────
  = TOTAL INCOME (taxable base)
  ─────────────────────────────────────────
  − Gross IRPEF (3 progressive brackets)
  + Employee tax deductions (art. 13 TUIR)
  + Dependent family deductions (spouse, children ≥ 21, others)
  + Additional tax wedge deduction (income €20K-€40K)
  ─────────────────────────────────────────
  = NET IRPEF
  ─────────────────────────────────────────
  − Regional IRPEF surtax
  − Municipal IRPEF surtax
  + Tax wedge allowance (income ≤ €20K, non-taxable)
  + Supplementary treatment (if applicable)
  ─────────────────────────────────────────
  = ANNUAL NET
  ÷ number of paychecks (13 or 14)
  = MONTHLY NET

  Separately: accrued TFR (does not pass through the payslip)
```

---

## Breakdown of each component

### 1. Employee INPS contributions

Social-security contributions paid by the worker fund their future pension.

| Sector | Rate | Notes |
|---------|----------|------|
| **Private** | 9.19% up to €55.448; 10.19% above | The threshold (first pensionable band) is updated annually by INPS. The extra 1% is set by art. 3-ter of Law 438/1992 |
| **Public** | 8.80% flat | Unified rate of the former INPDAP scheme (CTPS/CPDEL). No threshold |

**What is not included:** the employer's INPS contribution (~24-32%) is counted only in the employer cost estimate.

**Legal references:** Law 335/1995 (Dini reform), Law 438/1992 (additional 1% contribution), annual INPS circulars.

---

### 2. Fringe benefits

Fringe benefits are in-kind compensation (company car, meal vouchers, phone, housing) that the employer provides to the employee.

| Condition | Exemption threshold 2025-2027 |
|------------|---------------------------|
| Employee **without** dependent children | **€1.000** |
| Employee **with** dependent children | **€2.000** |

**Key rule:** if the total value of fringe benefits **exceeds** the threshold, the **entire amount** becomes taxable income (not just the excess). It is an all-or-nothing threshold, not a deduction.

**In the calculator:** the user enters the total value of annual fringe benefits. If above the threshold, the amount is added to the taxable base.

**References:** Art. 51, para. 3 TUIR; Law 207/2024 (2025 Budget Law), art. 1 paras. 390-391.

---

### 3. Supplementary pension fund

Contributions paid into supplementary pension schemes (negotiated funds, open funds, PIPs) are **deductible from total income** up to **€5.164,57/year**.

This directly reduces the IRPEF taxable base, generating a tax saving equal to the contribution × the taxpayer's marginal IRPEF rate.

**Example:** an employee with a RAL of €35.000 who pays €2.000 into the pension fund saves about €2.000 × 35% = €700 in IRPEF.

**What is not included in the calculator:** the employer's contribution to the pension fund (also deductible within the same cap).

**References:** Legislative Decree 252/2005, art. 8 para. 4.

---

### 4. IRPEF 2025

IRPEF, the personal income tax, is calculated by progressive brackets. Since 2024 (Legislative Decree 216/2023), confirmed structurally in 2025, there are 3 brackets:

| Bracket | Rate | On income up to | Cumulative tax |
|-----------|----------|-------------------|------------------|
| 1st | **23%** | €28.000 | €6.440 |
| 2nd | **35%** | €50.000 | €6.440 + €7.700 = €14.140 |
| 3rd | **43%** | above | €14.140 + 43% on the excess |

IRPEF is **progressive by bracket**: every additional euro is taxed at the rate of the bracket it falls into, not at the highest rate on the entire income.

**IRPEF taxable base** = RAL + taxable fringe benefits − employee INPS − deductible pension fund.

**References:** Art. 11 TUIR; Legislative Decree 216/2023; Law 207/2024.

---

### 5. Employee tax deductions

Deductions reduce gross IRPEF and are inversely proportional to income (those who earn less pay less tax).

| Total income | Deduction |
|---------------------|-----------|
| ≤ €15.000 | €1.955 |
| €15.001 – €28.000 | €1.910 + €1.190 × (28.000 − income) / 13.000 |
| €28.001 – €50.000 | €1.910 × (50.000 − income) / 22.000 |
| > €50.000 | €0 |

**€65 bonus:** for income between €25.001 and €35.000, €65 is added to the calculated deduction (Legislative Decree 216/2023).

**Constraint:** the deduction cannot exceed gross IRPEF (it does not generate a tax credit).

**References:** Art. 13 TUIR; Legislative Decree 216/2023 art. 1 para. 2.

---

### 6. Dependent family deductions

After the introduction of the Single Allowance (*Assegno Unico*, March 2022), payslip deductions concern only:

#### Dependent spouse (income ≤ €2.840,51)

| Taxpayer's income | Deduction |
|--------------------------|-----------|
| ≤ €15.000 | 800 − 110 × (income / 15.000) |
| €15.001 – €40.000 | €690 (with variations between €29K-€35.2K: see table in the code) |
| €40.001 – €80.000 | 690 × (80.000 − income) / 40.000 |
| > €80.000 | €0 |

#### Dependent children ≥ 21 years

- **€950** per child ≥ 21 with income ≤ €2.840,51 (or ≤ €4.000 if under 24)
- Phase-out: the deduction is reduced proportionally: × max(0, (95.000 − income) / 95.000)
- For children **< 21 years**: you receive the **Single Allowance** (paid by INPS, outside the payslip)

#### Other dependent family members

- **€750** each (parents, siblings, etc. living together with income ≤ €2.840,51)
- Phase-out: × max(0, (80.000 − income) / 80.000)

**References:** Art. 12 TUIR; Legislative Decree 230/2021 (Single Allowance).

---

### 7. Tax wedge bonus 2025

The 2025 Budget Law (Law 207/2024) made the reduction of the contribution wedge **structural**, turning it into a two-component tax mechanism:

#### A) Allowance (income ≤ €20.000) — non-taxable

For the lowest incomes, an allowance is paid, calculated as a percentage of employment income:

| Income | Percentage |
|---------|-------------|
| ≤ €8.500 | 7.1% |
| €8.501 – €15.000 | 5.3% |
| €15.001 – €20.000 | 4.8% |

This allowance **does not contribute to taxable income** (it is not taxed) and is added directly to the net.

#### B) Additional IRPEF deduction (income €20.001 – €40.000)

| Income | Deduction |
|---------|-----------|
| €20.001 – €32.000 | €1.000 fixed |
| €32.001 – €40.000 | €1.000 × (40.000 − income) / 8.000 |
| > €40.000 | €0 |

This deduction reduces net IRPEF (it is added to the employee tax deductions).

**What it replaced:** the 6-7% contribution cut on 2023-2024 payslips (which was temporary).

**References:** Law 207/2024, art. 1 paras. 4-9.

---

### 8. Supplementary treatment

The "supplementary treatment" (formerly the Renzi Bonus, formerly the €80 Bonus) is an IRPEF credit of **€1.200/year (€100/month)**.

It applies provided that:
- Total income is **≤ €15.000**
- Gross IRPEF is **higher** than the employee tax deductions (otherwise there is no IRPEF from which to "offset" the bonus)

**Interaction with the 2025 tax wedge:** for income ≤ €15.000, the tax wedge allowance (component A) is generally more advantageous than the supplementary treatment. In the calculator, when the tax wedge allowance is active, the supplementary treatment is not paid, to avoid a double benefit.

**References:** Art. 1 Decree-Law 3/2020 (converted into Law 21/2020).

---

### 9. Regional IRPEF surtax

Each Region/Autonomous Province applies a rate on the IRPEF taxable base. Some have a single rate, others use progressive brackets.

The calculator includes **all 21 regions** (including the Autonomous Provinces of Trento and Bolzano) with rates updated to 2025:

| Region | Type | Rate range |
|---------|------|----------------|
| Abruzzo | 3 brackets | 1.67% – 3.33% |
| Basilicata | Single | 1.23% |
| Calabria | Single | 1.73% |
| Campania | 4 brackets | 1.73% – 3.33% |
| Emilia-Romagna | 4 brackets | 1.33% – 3.33% |
| Friuli Venezia Giulia | 2 brackets | 0.70% – 1.23% |
| Lazio | 2 brackets | 1.73% – 3.33% |
| Liguria | 3 brackets | 1.23% – 3.23% |
| Lombardia | 4 brackets | 1.23% – 1.73% |
| Marche | 4 brackets | 1.23% – 1.73% |
| Molise | 3 brackets | 1.73% – 3.33% |
| Piemonte | 4 brackets | 1.62% – 3.33% |
| Puglia | 4 brackets | 1.33% – 1.85% |
| Sardegna | Single | 1.23% |
| Sicilia | Single | 1.23% |
| Toscana | 4 brackets | 1.42% – 3.33% |
| Trentino-AA (Bolzano) | 2 brackets | 1.23% – 1.73% |
| Trentino-AA (Trento) | 3 brackets | 0% – 1.73% |
| Umbria | 3 brackets | 1.23% – 3.33% |
| Valle d'Aosta | Single | 1.23% |
| Veneto | Single | 1.23% |

**Note:** Trento has a no-tax area up to €27.000 (0% rate).

**Sources:** Regional resolutions for tax year 2024/2025; QuantoPrendo.io, Money.it.

---

### 10. Municipal IRPEF surtax

Each Italian municipality sets its own rate (0% – 0.9% max) on the IRPEF taxable base.

The calculator includes the **main municipalities** for each region with preset rates, plus the option to enter a custom rate for municipalities not on the list.

**Examples:** Milan 0.8% · Rome 0.9% · Naples 0.9% · Florence 0.2% · Bologna 0.8% · Bolzano 0.1%.

**Sources:** Municipal resolutions; Fiscal Federalism Portal (MEF).

---

### 11. TFR — Severance pay

The TFR (*Trattamento di Fine Rapporto*) is a portion of pay that the employer **sets aside** annually and pays out at the end of the employment relationship. It **does not pass through the payslip** (it does not reduce the monthly net), but it is an important component of deferred income.

| Item | Formula |
|------|---------|
| Gross accrual | RAL / 13.5 (~7.41% of RAL) |
| INPS guarantee-fund contribution | − 0.50% of RAL |
| **Net accrual** | **~6.91% of RAL** |

**TFR destination:**
- Left in the company (for companies with < 50 employees)
- Paid into the INPS Treasury Fund (for companies with ≥ 50 employees)
- Transferred to a supplementary pension fund (the worker's choice)

**Taxation:** the TFR is subject to **separate taxation** (it is not cumulated with annual income). The rate applied is the average of the IRPEF rates of the last 5 years of work. In the calculator, we estimate this rate using the average IRPEF rate of the current year.

**References:** Art. 2120 of the Civil Code; art. 17 TUIR.

---

### 12. Employer cost

The "employer cost" is the total cost the employer bears for the employee. It is always significantly higher than the RAL.

| Component | Private | Public |
|-----------|---------|----------|
| RAL | 100% | 100% |
| Employer INPS | ~31% | ~24.2% |
| TFR accrual | ~7.4% | ~7.4% |
| INAIL | ~0.4% | — |
| **Estimated total** | **~138-140% of RAL** | **~131-132% of RAL** |

**Note:** the estimate is simplified. The real cost varies by sector, collective agreement (CCNL), INAIL risk class, presence of company welfare, etc.

---

## Available inputs

| Input | Description | Default |
|-------|-------------|---------|
| RAL | Gross Annual Salary | €30.000 |
| Paychecks | 13 (13th month) or 14 (+ 14th month) | 13 |
| Sector | Private or Public | Private |
| Region | 21 options (all Italian regions) | Lombardia |
| Municipality | Main cities per region + custom rate | Milan |
| Dependent spouse | Checkbox | No |
| Dependent children ≥ 21 | Number | 0 |
| Other dependent family members | Number | 0 |
| Dependent children < 21 | Checkbox (raises fringe threshold to €2.000) | No |
| Fringe benefits | Annual amount in € | €0 |
| Pension fund | Annual contribution in € | €0 |

---

## Generated outputs

1. **Annual net** and **monthly net** (÷ number of paychecks)
2. **Total withholdings** with percentage of RAL
3. **Accrued TFR** per year (gross and estimated net)
4. Estimated **employer cost**
5. **Visual bar** with the composition of the RAL
6. **Breakdown table** line by line with +/− signs
7. **IRPEF detail** by bracket (taxable base, rate, tax)
8. **Regional surtax detail** by bracket
9. **TFR detail** (accrual, INPS contribution, taxation)
10. **Tax wedge detail** (type, percentage, amount)
11. **Full list of assumptions** applied

---

## Simplifications and known limits

This is a **prototype** that covers the most common cases. Here is what was simplified and why:

| Simplification | Rationale |
|-----------------|-------------|
| **Single Allowance not included** | The Single Allowance for children < 21 is paid by INPS separately and does not pass through the payslip. Including it would require ISEE as an input |
| **Itemized deductions not included** | Medical expenses, mortgage interest, renovations, etc. are individual and cannot be predicted without the tax return (730) |
| **Simplified municipal surtax** | Some municipalities have progressive (non-flat) rates. We used the known values for the main provincial capitals |
| **TFR with estimated average rate** | Real separate taxation uses the IRPEF average of the last 5 years. Here we use the average rate of the current year |
| **Tax wedge/treatment interaction** | The supplementary treatment and the tax wedge allowance can interact in complex ways for income around €15K. We simplified: if the tax wedge is active, the treatment is not cumulated |
| **CCNL not differentiated** | Different collective agreements may include additional contributions (e.g. CIGS for companies with > 15 employees: +0.30%). We use the base rate |
| **Contribution ceiling not applied** | For new members enrolled after 1996 with a RAL > ~€120K there is a cap on INPS contributions. Not implemented |
| **Tax-return adjustment not included** | The calculator simulates the "steady-state" net, without refunds/charges from the income tax return |

---

## Project structure

```
calcolatore-ral-netto/
├── index.html    # The entire calculator (HTML + CSS + JS)
└── README.md     # This file
```

A deliberate choice: **zero dependencies**. The `index.html` file is self-contained and can be opened directly in the browser. No Node.js, npm, build tool, or server needed. This makes the prototype immediately verifiable and deployable to GitHub Pages with no configuration.

---

## How to run

### Locally
```bash
# Just open the file in the browser
open index.html
# or
python3 -m http.server 8080  # then visit http://localhost:8080
```

### Online
The project is hosted on GitHub Pages: **[→ Live demo](https://alessandro114.github.io/calcolatore-ral-netto/)**

---

## Main legal sources

- **TUIR** — Presidential Decree 917/1986 (Consolidated Income Tax Act)
- **Legislative Decree 216/2023** — IRPEF reform to 3 brackets
- **Law 207/2024** — 2025 Budget Law (structural tax wedge, fringe benefits 2025-2027)
- **Legislative Decree 252/2005** — Supplementary pensions
- **Legislative Decree 230/2021** — Universal Single Allowance
- **Decree-Law 3/2020** — Supplementary treatment
- **Art. 2120 Civil Code** — Severance pay (TFR)
- **INPS circulars** — Annual contribution rates
- **Regional and municipal resolutions** — IRPEF surtaxes
