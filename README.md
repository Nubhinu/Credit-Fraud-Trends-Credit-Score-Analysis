# Credit Scoring & Risk Analysis in Lending

An end-to-end analytics project on a synthetic loan-application dataset (20,000 applicants, 7 UK regions, April 2023 – April 2025). It covers a Python ETL pipeline, exploratory analysis, SQL risk analysis, and an interactive BI dashboard.

## Objective

Analyse risk factors, credit score behaviour, and default likelihood across a simulated lending portfolio, using the workflow a real credit-risk team would follow: prepare the data, explore it, query it, and visualise it.

## Project structure

| File | Phase | Description |
|---|---|---|
| `credit_scoring_dataset_raw_20000.csv` | Input | Original raw dataset, unmodified (20,000 rows, 20 columns) |
| `credit_pipeline.py` | 0 | ETL pipeline: extract, transform, validate, save |
| `cleaned_credit_data.csv` | 0 | Pipeline output (20,000 rows, 25 columns) for SQL, Tableau and Power BI |
| `credit_analysis.db` | 0 / 2 | SQLite database containing the table `credit_applications` |
| `phase2_sql_queries.sql` | 2 | The five required SQL analyses |
| `dashboard.html` | 3 | Interactive dashboard with live filters |
| `Credit_Scoring_Project_Report.docx` | All | Full write-up: method, code, results, insights |
| `README.md` | | This file |

## Requirements

- Python 3.9+
- `pandas` and `numpy` (`pip install pandas numpy`)
- `sqlite3` is part of the Python standard library
- A modern browser for `dashboard.html`. It loads Chart.js and Google Fonts from a CDN, so it needs an internet connection.

## How to run

### 1. Run the ETL pipeline

`credit_pipeline.py` reads a file called `raw_credit_data.csv` from the current directory. The raw file in this package has a longer name, so copy or rename it first:

```bash
cp credit_scoring_dataset_raw_20000.csv raw_credit_data.csv
python credit_pipeline.py
```

This regenerates `cleaned_credit_data.csv` and `credit_analysis.db` in the same folder. To use a different input path, edit `RAW_PATH` at the top of the script or call `run_pipeline(raw_path=...)`.

### 2. Run the SQL analysis

Open `credit_analysis.db` in any SQLite client (DB Browser for SQLite, DBeaver, VS Code SQLite extension) and run `phase2_sql_queries.sql`. Or from the command line:

```bash
sqlite3 credit_analysis.db < phase2_sql_queries.sql
```

The queries use SQLite syntax and port to MySQL, PostgreSQL and SQL Server with small changes (mainly `ROUND` and `CASE` casting).

### 3. Open the dashboard

Double-click `dashboard.html`. No server or BI licence is needed. The three filters (Region, Credit Risk Tier, Employment Years) update every KPI and chart live.

### 4. Tableau / Power BI

Connect directly to `cleaned_credit_data.csv`. The report (section 5.4) has a field-by-field mapping for each visual.

## Phase 0: the pipeline

| Function | What it does |
|---|---|
| `extract_credit_data()` | Loads the raw CSV into a DataFrame |
| `transform_credit_data(df)` | Fixes data types and adds engineered features |
| `validate_credit_data(df)` | Drops rows that break the data-quality rules |
| `save_credit_data(df)` | Writes the cleaned CSV and the indexed SQLite database |

**Engineered columns**

| Column | Definition |
|---|---|
| `debt_to_income_ratio` | `monthly_expense ÷ (annual_income / 12)` |
| `credit_risk_tier` | Poor (300–579), Fair (580–669), Good (670–739), Very Good (740–799), Exceptional (800–850) |
| `income_group` | `<30K`, `30K-60K`, `60K-90K`, `90K+` |
| `approved_flag` | 1 if `approval_status` is Approved, else 0 |
| `late_payment_band` | `0`, `1-2`, `3+` late payments |

**Validation rules:** income > 0, credit score 300–850, age 18–100, loan amount > 0, no duplicate `applicant_id`. All 20,000 rows passed.

## Phase 2: SQL questions answered

1. Top loan purposes requested
2. Approval rates by region and by employment type
3. Credit score trends vs. default history
4. Risk classification by late payments
5. Average loan size by income group

## Phase 3: dashboard contents

- KPI cards: applications in view, average credit score, default rate, approved loans
- Scatter: income vs. credit score, coloured by risk tier
- Bar chart: loan purposes
- Doughnut chart: employment status
- Filters: region, risk tier, employment years, with a reset button

The scatter plots a sample of up to 1,500 points for speed. KPIs and the other charts use every row that matches the filters.

## Headline results

| Metric | Value |
|---|---|
| Average credit score | 574.5 |
| Default rate (`defaulted_before`) | 10.17% (2,033 of 20,000) |
| Approved loans | 6,780 of 20,000 (33.9%) |
| Approval rate range across regions | 32.5% (Liverpool) to 35.8% (Manchester) |
| Loan purposes | Evenly split, 19.5%–20.5% each |

## Assumptions

- **Debt-to-income ratio:** the dataset has no monthly debt-payment column, so `monthly_expense` is used as a proxy for debt service.
- **Risk tiers:** standard FICO-style bands. They are unequal in width, which is why the Poor tier holds about 51% of applicants.
- **Income groups:** fixed bands chosen to give reasonably even group sizes across the $15K–$120K income range.
- **Age validation:** the 18–100 bound is a general working-adult range. The data itself spans 21–64.

## Important note on the data

This is a synthetic dataset. Credit score, income and employment years are close to uniformly distributed (their means and standard deviations match a uniform draw almost exactly), and they are largely independent of the outcome fields (approval, default, risk flag). As a result:

- Approval rates barely vary by region or employment type.
- Prior defaulters score only about 8 points lower on average (567.7 vs. 575.3).
- Late-payment count does not raise the risk-flag or prior-default rate.

These are properties of how the data was generated, not findings about real lending. Use the project to demonstrate the ETL, SQL and BI workflow rather than to draw lending-strategy conclusions. The report discusses this in section 6.

## Data dictionary

The full dictionary for the 20 original and 5 engineered columns is in the appendix of `Credit_Scoring_Project_Report.docx`.

