# Descriptive Statistics: Let AI Help You Write Publication-Ready Descriptive Statistics Tables

When a reviewer opens your paper, what's the first thing they look at?

Not the regression results, not the robustness checks — it's **Table 1**, the descriptive statistics table.

This table tells the reader: what your data looks like, whether the sample is reasonable, and whether there are any issues with the variables. If Table 1 doesn't pass muster, no one will believe the regressions that follow, no matter how elegant they are.

But honestly, descriptive statistics itself isn't hard — what's hard is doing it **correctly, comprehensively, and to publication standard**. Whether to log-transform variables, how to handle outliers, how to present categorical variables... these are all details, but details determine the first impression of your paper.

Today I'll share a workflow for AI-assisted descriptive statistics, taking you step by step from raw data to a publication-ready Table 1.

## What a Publication-Ready Descriptive Statistics Table Looks Like

Let's start with the goal. A Table 1 in top economics and management journals typically includes the following columns:

| Column | Description | Required? |
|---|---|---|
| Variable Name | Bilingual (Chinese/English), logically grouped | Required |
| N | Observations (non-missing count) | Required |
| Mean | Mean | Required |
| SD | Standard deviation | Required |
| Min | Minimum | Required |
| Max | Maximum | Required |
| P25 / Median / P75 | Quantiles | Recommended |
| Skew / Kurt | Skewness / Kurtosis | Optional |

**Key convention**: Arrange variables in logical groups — dependent variable → key independent variables → control variables → instrumental variables, not alphabetically.

## Generate Descriptive Statistics Code with AI in One Step

Here's the prompt, ready to copy and use:

> I have a Stata dataset with the following variables:
> - Dependent variable: ln_wage (log wage)
> - Key independent variables: edu_year (years of education), train (received training, 0/1)
> - Control variables: age (age), gender (gender, 0/1), experience (years of experience), firm_size (firm size)
> - Data characteristics: Panel data, firm-year
>
> Please generate Stata code that:
> 1. Outputs a three-line formatted descriptive statistics table
> 2. Groups variables in the order: Dependent → Key Independent → Control, with a blank row between groups
> 3. Includes N, Mean, SD, Min, Max, P50
> 4. Exports to Word using esttab
> 5. Also checks for extreme values that may need Winsorization

The AI will give you a complete block of Stata code that looks something like this:

```stata
* 变量标签
label var ln_wage "对数工资"
label var edu_year "受教育年限"
label var train "是否参加培训"
label var age "年龄"
label var gender "性别"
label var experience "工作年限"
label var firm_size "企业规模"

* 定义变量组
global depvar "ln_wage"
global keyvar "edu_year train"
global ctrlvar "age gender experience firm_size"

* 描述统计
estpost summarize ${depvar} ${keyvar} ${ctrlvar}, detail
esttab using "desc_stats.rtf", ///
    cells("count(fmt(%9.0fc)) mean(fmt(%9.3f)) sd(fmt(%9.3f)) min(fmt(%9.3f)) p50(fmt(%9.3f)) max(fmt(%9.3f))") ///
    noobs nonumber nomtitle title("描述性统计") replace
```

Note: Always run AI-generated code yourself to verify. Small things like variable names, labels, and grouping order are easy to get wrong.

## Four Common Pitfalls — Let AI Help You Catch Them

### Pitfall 1: Large Disparities in Variable Magnitudes

If your descriptive statistics table shows one variable with a mean of 0.3 and another with 85,000 — putting them together conveys little information and makes regression coefficients hard to interpret.

**AI's suggestion**: Apply natural log transformation to large-valued variables (e.g., `ln_asset = ln(asset)`), or standardize them (`x_std = (x - mean) / sd`).

### Pitfall 2: Outliers Distorting Summary Statistics

Firm assets with a mean of 8 billion but a median of 500 million — a classic right-skewed distribution where a few giants inflate the mean.

**AI's suggestion**: Apply 1% or 5% two-sided Winsorization to continuous variables:

```stata
winsor2 asset, cuts(1 99) replace
```

### Pitfall 3: Zero or Negative Values Breaking Log Transformation

`ln(x)` requires x > 0. If a variable contains zeros (e.g., many firms have zero R&D expenditure), applying log directly will drop those observations.

**AI's suggestion**: Use `ln(x+1)` or `asinh(x)` (inverse hyperbolic sine transformation); the latter is zero-friendly and approximates the log function.

### Pitfall 4: Treating Categorical Variables as Continuous

Variables like gender (0/1) or industry code (1–18) — reporting a mean and standard deviation is meaningless.

**AI's suggestion**: For 0/1 variables, only report the frequency (the mean is the proportion). For multi-category variables, create a separate frequency distribution table.

## AI-Assisted Interpretation of Descriptive Statistics: Three Real-World Examples

### Case A: Large-Firm Bias

```
Variable        N     Mean      SD      Min     P50      Max
Total Assets  5,320  80.2亿   210亿   0.3亿   5.1亿  3200亿
```

Mean of 8 billion vs. median of 500 million — a 16-fold difference. This indicates the sample is severely right-skewed, with a few large firms dominating the mean. When reviewers see this, they will ask: **Are your regression results driven by large firms?** You'll need to follow up with subsample regressions or Winsorization.

### Case B: Insufficient Variation

```
Variable         N    Mean     SD     Min    Max
Industry HHI  2,100   0.03   0.01   0.01  0.05
```

The standard deviation is only 0.01 — the variable has almost no variation. Using it as a key independent variable, the regression coefficient will almost certainly be insignificant — not because there's no effect, but because the data reveals no difference. **Either change the variable or change the sample.**

### Case C: Data Entry Errors

```
Variable    N     Mean      SD      Min     Max
Age      3,800    35.2     12.8     18     999
```

A maximum of 999 is clearly a missing value code from data entry. In Stata, replace it with `.`:

```stata
replace age = . if age > 100
```

AI can easily catch these issues for you — prompt: **"Please check the following descriptive statistics and point out possible outliers and data entry errors."**

## One-Click Workflow: From Raw Data to Publication-Ready Table 1

Let's connect the steps above:

**Step 1**: Raw data → Have AI check variable types and distinguish continuous / categorical / 0-1

**Step 2**: AI provides Winsorization / log transformation / variable conversion suggestions

**Step 3**: AI generates Stata/R code to output descriptive statistics in one click

**Step 4**: Paste the results back into AI and have it interpret them — identifying outliers, right-skewed distributions, and insufficient variation

**Step 5**: AI helps you write a paragraph for the "Data and Variables" section of your paper:

> Table 1 reports the descriptive statistics of the main variables. The sample firms' average log wage is 10.83 (approximately 51,000 yuan), with a standard deviation of 0.76, indicating substantial variation in wage levels. The mean years of education is 14.2, and approximately 35% of the sample has participated in vocational training...

## Quick Reference: Tools

| Tool | Purpose | Command/Package |
|------|------|---------|
| Stata | Export three-line tables | `esttab`, `outreg2` |
| Stata | Winsorize | `winsor2` |
| R | Descriptive statistics tables | `gtsummary::tbl_summary()`, `modelsummary::datasummary()` |
| Python | Quick statistics | `df.describe()` + `tabulate` |
| AI | Code generation + interpretation | DeepSeek / GPT-4o |

---

Descriptive statistics are not a mere formality — they are your first conversation with the reviewer. Use AI to make this table correct, comprehensive, and thorough, and the analyses that follow will be persuasive.

Next time we'll talk about **data cleaning**: the AI workflow for handling missing values, outliers, and duplicate observations.
