Part 1 — Data Modeling and SQL (Lending Club 2007–2015)
The goal is to make the per‑grade default rate easy, fast, and consistent by standardizing parsing and a single default flag in a reusable view. The metric is defined as COUNT('Charged Off') / COUNT('All Loans'), so the “default” flag must be strictly tied to the “Charged Off” status.

Centralize a standardized default flag and normalize key columns from the raw Kaggle CSVs (or an external table over those CSVs), restricted to the 2007–2015 window.

Schema

Column	      Type	   Description
loan_id	      STRING	Unique loan identifier if present in the raw file (use id); otherwise retain null.
member_id     STRING	Anonymized borrower identifier from raw data.
grade	      STRING	Lending Club credit grade A–G.
sub_grade	  STRING	Subgrade A1–G5.
loan_status   STRING	Raw loan status string (e.g., Fully Paid, Charged Off, Current).
is_default	  INT64	    1 if loan_status = 'Charged Off', else 0.
issue_date	  DATE	    Parsed origination month from issue_d (format Mon-YYYY).
term_months	  INT64	    Number of months parsed from term (e.g., 36 or 60).
int_rate	  FLOAT64	Annual interest rate as decimal (e.g., 0.1399).
loan_amount	  NUMERIC	Requested loan amount (from loan_amnt).
funded_amount NUMERIC	Funded amount (from funded_amnt).


Why this design over raw CSV queries

Consistent “default” logic: Standardizes the strict business definition (Charged Off only) so analysts don’t re‑implement or accidentally widen the definition.

Clean types once: Parses dates, terms, and interest rate one time for everyone, removing repeated wrangling and potential type errors.

Simple and fast metric queries: The default‑rate query reduces to a small GROUP BY over a canonical view filtered to the correct time window.

BigQuery SQL

Create the clean analytics view

sql
-- Create a clean, analysis-ready view
CREATE OR REPLACE VIEW `project.dataset.loans_clean` AS
SELECT
  CAST(id AS STRING) AS loan_id,
  CAST(member_id AS STRING) AS member_id,
  CAST(grade AS STRING) AS grade,
  CAST(sub_grade AS STRING) AS sub_grade,
  CAST(loan_status AS STRING) AS loan_status,

  -- Strict default per prompt: ONLY 'Charged Off'
  CASE WHEN loan_status = 'Charged Off' THEN 1 ELSE 0 END AS is_default,

  -- Parse and normalize critical fields
  PARSE_DATE('%b-%Y', issue_d) AS issue_date,
  CAST(REGEXP_EXTRACT(term, r'\d+') AS INT64) AS term_months,
  CAST(REPLACE(int_rate, '%', '') AS FLOAT64) / 100.0 AS int_rate,

  -- Money fields as NUMERIC to avoid float rounding
  CAST(loan_amnt AS NUMERIC) AS loan_amount,
  CAST(funded_amnt AS NUMERIC) AS funded_amount
FROM `project.dataset.raw_loans`
WHERE
  -- Keep only rows with a valid issue date and restrict to 2007–2015
  REGEXP_CONTAINS(issue_d, r'^[A-Za-z]{3}-\d{4}$')
  AND PARSE_DATE('%b-%Y', issue_d) BETWEEN DATE '2007-01-01' AND DATE '2015-12-31';


Metric: default rate by grade

sql
-- Default rate = COUNT('Charged Off') / COUNT('All Loans')
SELECT
  grade,
  COUNT(*) AS all_loans,
  SUM(is_default) AS charged_off_loans,
  SAFE_DIVIDE(SUM(is_default), COUNT(*)) AS default_rate
FROM `project.dataset.loans_clean`
GROUP BY grade
ORDER BY grade;

Notes 

If the raw dataset contains statuses like “Does not meet the credit policy. Status: Charged Off,” they will not be counted as defaults because the definition is strictly loan_status = 'Charged Off'; 

Optional steps to include are to  add a WHERE clause to exclude such policy‑exception statuses entirely from the denominator to keep populations consistent across vintages.
