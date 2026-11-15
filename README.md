# E-Commerce Sales and Customer Behavior Analytics

SQL and Python analysis of a Brazilian e-commerce dataset. I used MySQL to query the data and Python to clean, analyse and visualise the results.

I built this to get real practice with SQL and Python on something messier than a textbook dataset. I am studying Data Science at TU Chemnitz, and I wanted a project where I had to set up the database myself, write the queries, and then decide what the numbers actually meant.

So I imported 1.45 million rows across seven tables into MySQL and worked through fifteen business questions. What I did not expect was how much of the time went into checking the data before trusting it. Early on I plotted monthly orders for 2018 and it looked like sales collapsed in September. They did not. The dataset just stops in October. That one chart is the reason I ended up writing a separate data quality audit before finishing the analysis.

## Dataset

The data comes from the Olist Brazilian E-Commerce Public Dataset on Kaggle:
https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce

Seven CSV files, 1,451,627 rows in total:

| Table | Rows |
|---|---:|
| customers | 99,441 |
| geolocation | 1,000,163 |
| order_items | 112,650 |
| orders | 99,441 |
| payments | 103,886 |
| products | 32,951 |
| sellers | 3,095 |

The CSV files are not in this repo. Download them from the Kaggle link above and check the licence there before reusing the data.

All money values are in Brazilian Real (BRL).

## What I found

| Question | Answer |
|---|---|
| Orders placed in 2017 | 45,101 |
| Top category by item sales | Health Beauty, R$1,258,681.34 |
| Share of orders paid in instalments | 51.46% |
| Biggest customer state | Sao Paulo, 41,746 of 99,441 records |
| Price vs purchase frequency (Pearson r) | -0.0321, basically no relationship |
| Top seller by item sales | R$229,472.63 |
| Payment value, Jan to Aug 2018 vs 2017 | +136.98% |
| Customers who bought again within 6 months | 2.88% |

Two of these are worth more than the rest.

The 2.88% retention rate is the number I would take to a business. Almost nobody comes back. For a marketplace that is a real problem, and it probably matters more than any of the sales figures above it.

The +136.98% growth looks great until you ask what is behind it. The platform was still scaling up through early 2017, so a large part of that jump is the business getting bigger rather than existing customers spending more. I kept the comparison to the same calendar months in both years so at least the seasonality lines up.

## Questions I answered

1. Which cities do customers come from?
2. How many orders were placed in 2017?
3. Which product categories sell the most?
4. What share of orders are paid in instalments?
5. How are customers spread across states?
6. How did monthly order volume change through 2018?
7. What is the average basket size per city?
8. How much does each category contribute to total sales?
9. Is product price linked to how often something sells?
10. Which sellers sell the most?
11. How does one repeat customer's spending move over time (3 order moving average)?
12. How do cumulative monthly payments grow?
13. How does Jan to Aug payment value compare year over year?
14. How many first time customers buy again within six months?
15. Who were the top 3 spenders each year?

SQL used: joins, CTEs, GROUP BY, HAVING, CASE WHEN, and window functions (ROW_NUMBER, DENSE_RANK, LAG, and a moving average with ROWS BETWEEN).

## Data quality checks

Before trusting any of the numbers I ran a separate audit notebook. What came out of it:

- All five table relationships checked out. No orphan records.
- customer_id is unique per row, but there are only 96,096 real people in the data. The customers table has one row per order, not per person. Anything about customers in this project uses customer_unique_id instead. Getting this wrong would have made the retention rate come out near zero.
- One delivered order from September 2016 has no payment record at all.
- Nine payment records have a value of zero, and two credit card records show zero instalments.
- 610 products are missing their category and description.
- Fourteen orders have a status and delivery date combination that does not make sense.
- Dates came in as TEXT rather than DATETIME.

I did not delete or change anything in the raw tables. Problems are written down, not cleaned away.

### The date gap

September 2018 shows 16 orders and October shows 4, after months of around 6,500 each. On a chart that looks like the business fell off a cliff.

It is not a crash. The data just ends on 17 October 2018. So I checked the full date range, limited the 2018 trend chart to January through August, and said why on the chart itself. Plotting the whole year would have told a story that is not in the data.

## Run the import script

You need Python 3.10 or newer, MySQL and Jupyter.

```bash
pip install -r requirements.txt
```

Then:

1. Create an empty MySQL database called `ecommerce`.
2. Download the seven CSV files from Kaggle into one folder.
3. Run the import script and point it at that folder.
4. Open `Data_Quality_Audit.ipynb` first, then `Ecommerce_Analytics.ipynb`.

The notebooks ask for your MySQL password when they connect, so it is never stored in the code. Both notebooks only read from the database.

## Limitations

- 2016 is only partly covered and 2018 stops in October, so full year comparisons are not reliable.
- Sales figures are the sum of item prices. Freight is not included and this is not profit.
- Correlation is not causation. The price analysis says the two are unrelated in a straight line, nothing more.
- The geolocation outlier check uses a rough box around Brazil, not real borders.
- Everything here is based on this snapshot of the data.

## What I would do next

- Add indexes and proper data types. The geolocation table alone has a million rows and the joins are slow without them.
- Build a cleaned layer instead of querying the raw import every time.
- Work out why retention is so low. Which categories, which regions, and whether delivery time has anything to do with it.

## Tools

MySQL, Python, Pandas, NumPy, Matplotlib, Seaborn, Jupyter Notebook.

---

Muhammad Zubair
