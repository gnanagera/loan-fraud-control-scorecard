# Loan Fraud Control Scorecard

I built this project to practice the kind of reporting a bank's risk team does: take raw data, check that it can be trusted, turn it into a few risk indicators with thresholds, and show the results in a dashboard. The data is a loan application and transaction fraud dataset, and everything is done in SQL Server and Power BI.



## The question

How much fraud is showing up in loan applications each month, and how much of it is the bank's controls missing?

The dataset labels each fraudulent application as either detected or undetected, which makes the second part measurable. That is the part I found most interesting, because it shows how well the controls are working and not just how much fraud there is.

## The data

Two CSV files from a fraud detection dataset:

| File | Rows | Notes |
|---|---|---|
| `loan_applications.csv` | 50,000 | June 2022 to June 2025, about 18,000 customers, 1,026 fraud cases |
| `transactions.csv` | 50,000 | January 2022 only, 502 fraud cases |

The CSVs are not in this repo. They contain address-style personal fields and are large, so I left them out. You can load your own copy to reproduce the work.

## What I did

1. **Loaded the raw files** into SQL Server staging tables and checked the row counts and fraud totals against the CSVs (50,000 rows each, 1,026 and 502 fraud cases).
2. **Profiled the data.** I looked for duplicate IDs, blank values, date ranges, and the mix of loan statuses.
3. **Cleaned it** into typed tables (`loans_clean`, `txn_clean`) and left the raw staging tables untouched.
4. **Wrote data quality checks** that log failures into an exceptions table.
5. **Built the KRI view** in SQL with monthly values and Green/Amber/Red status.
6. **Built a fraud-by-type view** that splits each fraud type into detected and undetected.
7. **Made a Power BI dashboard** on top of those views and checked a month of numbers against the SQL results.

## The two indicators

| KRI | How it's calculated | Green | Amber | Red |
|---|---|---|---|---|
| Fraud rate | fraud applications / all applications in the month | under 2% | 2% to 3% | over 3% |
| Undetected share | undetected fraud / all fraud in the month | under 40% | 40% to 55% | over 55% |

I picked these thresholds myself by looking at how the monthly values were spread out. In a real bank, a risk committee would set and approve them, so treat them as examples.

I only used full months (July 2022 to May 2025, 35 months). The first and last months in the file are partial, and a partial month would distort the rates.

## What I found

- Overall, **2.05%** of applications were fraud (1,026 of 50,000).
- Only **52.8%** of the fraud was detected (542 cases). The other 484 cases got through.
- **September 2022** was the worst month for fraud rate at 3.27%, which is over my red threshold.
- Some months had a very high share of undetected fraud. July 2022 was 68.75%, with more than two-thirds of that month's fraud slipping past the controls.
- [Add 1-2 findings from your own dashboard here, for example how many months were red on each KRI.]
- The four fraud types (synthetic identity, income misrepresentation, loan stacking, transaction laundering) each account for roughly a quarter of the cases.

## Data quality checks

I wrote eight rules. Five passed with zero failures and three produced exceptions worth reporting:

| Rule | Check | Result |
|---|---|---|
| DQ01 | Duplicate application IDs | 0 |
| DQ02 | Credit score missing or outside 300 to 900 | 0 |
| DQ03 | Applicant under 18 or age missing | 0 |
| DQ04 | Debt-to-income ratio doesn't match EMIs and income | 0 |
| DQ05 | Fraud flag and fraud type disagree | 0 |
| DQ06 | Loan requested above 60x monthly income | 1,310 |
| DQ07 | Approved loan to an unemployed applicant | 6,866 |
| DQ08 | Transaction leaves a zero balance | 3,762 |

DQ06 and DQ07 look like real policy questions, since a lot of large loans were approved for applicants with no job. In a real setting I would send those to the credit policy owner. DQ08 is low severity and I'm only reporting it for information.

## Limitations

- **The two files don't overlap in time.** Transactions are all from January 2022 and applications start in June 2022. The files share customer IDs, but I can't say a transaction led to a later loan decision, so I kept the two analyses separate.
- **The data looks synthetic.** Fraud rates are almost the same across loan types, employment status, and credit score bands, so I did not try to claim any fraud "drivers", and I did not build a prediction model.
- **The fraud labels come with the dataset.** I did not verify them independently.
- **The thresholds are my own assumptions.**

## Repository layout

```
sql/        T-SQL scripts, numbered in the order I ran them
powerbi/    Power BI report (.pbix)
docs/       Dashboard screenshots, issue log, data dictionary
README.md
```

## How to reproduce it

1. Create a SQL Server database and load both CSVs into `stg_loan_applications` and `stg_transactions`.
2. Run the scripts in `sql/` in order (01 through 06).
3. Open `powerbi/fraud_scorecard.pbix` and point the data source at your own server and database.

## What I would add next

- A control-testing sample: pull a random set of applications and check them against the approval rules
- More KRIs, such as approval rate for high-risk applicants
- Alert rules for the transactions data

## About me

[Your name] | Data Engineer | [LinkedIn URL]
