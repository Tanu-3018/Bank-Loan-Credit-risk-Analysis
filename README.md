# Bank-Loan-Credit-risk-Analy
# Bank Loan Credit Risk Analysis

**End-to-end analysis of 38,576 loan applications (2021): data cleaning in Python, KPI reporting in SQL Server, and a 3-page interactive Power BI dashboard.**

> **Core question:** which loans go bad, and why?

![Overview dashboard](Dashboards/overview.png)

---

## Table of Contents
1. [Project Summary](#project-summary)
2. [Key Findings](#key-findings)
3. [Tools Used](#tools-used)
4. [Repository Structure](#repository-structure)
5. [Dataset](#dataset)
6. [Methodology](#methodology)
7. [Dashboard Pages](#dashboard-pages)
8. [Recommendations](#recommendations)
9. [Limitations](#limitations)
10. [How to Reproduce](#how-to-reproduce)
11. [Author](#author)

---

## Project Summary

A bank wants to understand the health of its loan book and spot where it is losing money. This project answers that in three layers:

| Layer | Tool | What it does |
|---|---|---|
| Clean + explore | Python (pandas, matplotlib, seaborn) | Validates the data, fixes types and dates, and finds which borrower and loan groups default most |
| Report KPIs | SQL Server (T-SQL) | Computes headline KPIs, month-over-month changes, good vs bad loan splits and risk breakdowns |
| Present | Power BI | Three-page dashboard with filters for State, Grade, Purpose, Term and Good vs Bad loan |

**Definitions used throughout**
- **Good loan** = `Fully Paid` or `Current`
- **Bad loan** = `Charged Off`
- **Bad Loan %** = Charged Off loans in a group / all loans in that group x 100
- **Current month** = December 2021, **Previous month** = November 2021

---

## Key Findings

| # | Finding | Number |
|---|---|---|
| 1 | Overall loan quality | **86.18%** good, **13.82%** bad (5,333 of 38,576 loans) |
| 2 | Money lost on bad loans | Funded **65.5M**, recovered **37.3M** (56.9%), so about **28.2M not recovered** |
| 3 | Grade is the strongest risk signal | Bad loan % climbs steadily from **5.7% (Grade A)** to **31.3% (Grade G)** |
| 4 | Term matters a lot | 60-month loans default at **22.3%** vs **10.7%** for 36-month loans (about 2x) |
| 5 | Riskiest purpose | **Small business** loans at **25.6%**; safest are wedding (9.3%) and major purchase (9.8%) |
| 6 | Debt-to-Income (DTI) | Bad loan % rises from **12.0%** (DTI below 10%) to **15.6%** (DTI 20% and above) |
| 7 | Loan size | Biggest 25% of loans (15K to 35K) default at **17.4%** vs about 12 to 13% for smaller loans |
| 8 | Business growth | Applications grew from **2.3K (Jan)** to **4.3K (Dec)**; December vs November: applications **+6.9%**, funded amount **+13.0%** |
| 9 | A surprising result | **Verified** borrowers default more (15.7%) than **Not Verified** (12.2%), which needs further study |

**Headline KPIs (all of 2021)**

| KPI | Value |
|---|---|
| Total loan applications | 38,576 |
| Total funded amount | 435,757,075 |
| Total amount received | 473,070,933 |
| Average interest rate | 12.05% |
| Average DTI | 13.33% |

**Pricing vs risk:** the bank already charges more for riskier grades (median rate 7.49% for Grade A to 21.14% for Grade G), but the default rate grows faster than the interest rate, so the higher price may not cover the extra loss.

---

## Tools Used

- **Python:** pandas, NumPy, matplotlib, seaborn (Jupyter Notebook)
- **SQL Server:** T-SQL (SSMS)
- **Power BI Desktop:** KPI cards, drill filters, page navigation, DAX measures
- **Git / GitHub**

---

## Repository Structure

```
bank-loan-credit-risk-analysis/
│
├── README.md
├── bank_loan_cleaning_and_eda.ipynb   # Python: cleaning + EDA + SQL cross-check
├── SQLQuery.sql                       # SQL Server: cleaning checks, KPIs, risk analysis
├── financial_loan_clean.csv           # Cleaned dataset (38,576 rows, 24 columns)
└── Dashboards/
    ├── summary.png
    ├── overview.png
    └── details.png
```

---

## Dataset

- **Size:** 38,576 loan applications x 24 columns
- **Period:** loans issued January to December 2021
- **Grain:** one row per loan application

| Group | Columns |
|---|---|
| Loan | `id`, `loan_amount`, `term`, `int_rate`, `installment`, `grade`, `sub_grade`, `purpose`, `loan_status`, `application_type` |
| Borrower | `member_id`, `emp_title`, `emp_length`, `home_ownership`, `annual_income`, `dti`, `verification_status`, `total_acc`, `address_state` |
| Dates and payments | `issue_date`, `last_credit_pull_date`, `last_payment_date`, `next_payment_date`, `total_payment` |

---

## Methodology

### 1. Data cleaning (Python and SQL)
- Confirmed **38,576 records** and **no duplicate loan IDs**
- Only `emp_title` had missing values (**1,438 rows**), filled with `"Not Available"`
- Removed leading spaces in `term` (`" 36 months"` became `"36 months"`) so grouping and filtering work correctly
- Converted the four date columns from text to real dates. Raw dates are **day-month-year**, so the format was set explicitly to avoid swapping day and month
- Sanity checks: no loan amount below 1, no negative payments, interest rate within 0 to 1, DTI not above 1, and only 3 loan status values
- Kept income outliers (578 customers above 200K) because they are real customers, and capped only the chart view at the 99th percentile

### 2. Exploratory data analysis (Python)
- Univariate views of loan status, loan amount, interest rate, DTI, income, grade, purpose, term and home ownership
- Bad loan % compared across grade, term, purpose, home ownership, verification status, DTI band and loan size quartile, each against the overall average (13.82%)
- Interest rate by grade, money recovered on bad loans, monthly trend, and a correlation heatmap

### 3. SQL analysis (SQL Server)

| Part | Content |
|---|---|
| 0 | Load and clean: row count, duplicates, nulls, status values, trim `term`, date type |
| 1 | Summary KPIs with current month vs previous month (MTD / MoM) |
| 2 | Good loan KPIs |
| 3 | Bad loan KPIs |
| 4 | Loan status grid (all months and current month) |
| 5 | Breakdowns by month, state, term, employment length, purpose and home ownership |
| 6 | Filter example |
| 7 | Own risk analysis: bad loan % by grade, term, purpose, home ownership, verification, DTI band, and money lost |

The Python results were **cross-checked against SQL** and match exactly (applications, funded amount, amount received, good % and bad %).

### 4. Power BI dashboard
The cleaned CSV is loaded into Power BI. A shared navigation menu links the three pages, and slicers filter every page.

---

## Dashboard Pages

| Page | Purpose |
|---|---|
| **Summary** | KPI cards with month-over-month change, good vs bad loan split, and a loan status table |
| **Overview** | Trend by month, top 10 states by funded amount, term split, employment length, purpose and home ownership |
| **Details** | Loan-level table (ID, purpose, home ownership, grade, issue date, amount, rate, installment, amount received) |

| Summary | Details |
|---|---|
| ![Summary](Dashboards/summary.png) | ![Details](Dashboards/details.png) |

---

## Recommendations

1. **Be careful with 60-month and small business loans.** Ask for additional checks or a higher interest rate.
2. **Tighten approval for Grade E, F and G.** Their bad loan rate is about 25% to 31%.
3. **Set a clear DTI limit and a cap on very large loans** for riskier grades.
4. **Review the income verification process.** Verified customers defaulting more is counter-intuitive and may point to a process issue or to the bank verifying customers it already considers risky.
5. **Review pricing for high-risk grades.** Default rates rise faster than interest rates, so the current premium may not cover expected losses.

---

## Limitations

- Covers **one year (2021)** only, so seasonality and long-term trends cannot be confirmed.
- **Current loans (1,098)** are counted as good but are not finished, so the true bad loan rate may be slightly higher.
- Findings show **associations, not causes** (for example, longer terms default more, but the term itself is not proven to be the reason).
- Some groups are very small (for example home ownership `OTHER` = 98 loans, `NONE` = 3 loans) and should not be over-interpreted.

---

## How to Reproduce

**Python**
```bash
git clone https://github.com/Tanu-3018/bank-loan-credit-risk-analysis.git
cd bank-loan-credit-risk-analysis
pip install pandas numpy matplotlib seaborn jupyter
jupyter notebook bank_loan_cleaning_and_eda.ipynb
```
The notebook starts from the raw `financial_loan.csv` and writes `financial_loan_clean.csv`. To skip cleaning, load `financial_loan_clean.csv` directly.

**SQL Server**
1. In SSMS, import `financial_loan_clean.csv` as a table named `bank_loan_data` (Tasks > Import Flat File).
2. Run `SQLQuery.sql` part by part. Expected values are written in the comments next to each query.

**Power BI**
Load `financial_loan_clean.csv` (or connect to the SQL Server table) and build the visuals described above.

---

## Author

**Tanupriya**
GitHub: [Tanu-3018](https://github.com/Tanu-3018)
