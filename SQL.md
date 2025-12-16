# SQL

## Table of Content

- [SQL](#sql)
  - [Table of Content](#table-of-content)
  - [Basic commands](#basic-commands)
  - [Medium commands](#medium-commands)
  - [Complicated commands](#complicated-commands)
  - [Cohort analysis](#cohort-analysis)
  - [Funnel analysis](#funnel-analysis)

## Basic commands

Select, Where and Having (difference is that having filters after having retrieved data, it is used after a group by). Having is often used after a sum or mean.

Limit, order by to sort and limit values. **WARNING**: some sql types rather use Top than Limit.

Count, sum, min, max, avg and group by to aggregate values.

Union is concatenation in SQL.

UPPER, LOWER, Distinct, is null, coalesce to clean data. coalesce returns the first non null value in list.

## Medium commands

```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    WHEN conditionN THEN resultN
    ELSE result
END; 
```

With ... as () statement to have intermediate results (Common Table Expressions).

You have window functions to do operations on a subset:

```sql
SELECT Name, Age, Department, Salary, 
       AVG(Salary) OVER( PARTITION BY Department) AS Avg_Salary
 FROM employee

/*
Rank function counts equality as two different values and increases rank by as many values.
Dense_Rank counts rank on value numbers.
Row_Number gives row number without looking at rank. If you order by, you get row number.
*/
SELECT Name, Department, Salary,
       RANK() OVER(PARTITION BY Department ORDER BY Salary DESC) AS emp_rank
FROM employee;
```

There are some data functions that can be useful: Date_Trunc, Extract and Date_diff are the more useful.

## Complicated commands

Careful with performances, avoid select *, understand tradeoffs of denormalization, CTE, temporary tables, indexes...

Advanced windowing for time series data with LAG, LEAD, Rolling averages.

```sql
SELECT id, 
       start_date, 
       end_date, 
       DATEDIFF(LEAD(start_date) OVER (ORDER BY start_date), end_date) + 1 AS no_of_days 
FROM events;

SELECT 
    c_id, 
    start_date, 
    end_date, 
    LAG(end_date) OVER (ORDER BY start_date) AS prev_end_date,
    DATEDIFF(start_date, LAG(end_date) OVER (ORDER BY start_date)) AS gap_days
FROM contest;
```

**FULL OUTER JOIN** vs **CROSS JOIN**. Cross join takes the whole combinations (we always end up with [number of entries of table 1] * [number of entries of table 2]). In a FULL OUTER JOIN, we join on lines that correspond. If no line corresponds, we add NULLs to the column where we didn't find values.

**Grouping sets** when you need to have multiple levels of grouping:

```sql
SELECT
    Region,
    Product,
    SUM(Amount) AS TotalSales
FROM
    Sales
GROUP BY
    GROUPING SETS (
        (Region, Product),  -- Grouping Set 1: Total for each Region/Product combination
        (Region),           -- Grouping Set 2: Subtotal for each Region
        ()                  -- Grouping Set 3: Grand Total (empty set)
    )
ORDER BY
    Region, Product;
```

NOTE: there are some advanced grouping functions like STDDEV for standard deviation, VARIANCE, MEDIAN, PERCENTILE...

```sql
SELECT 
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY order_amount) AS median_order,
    PERCENTILE_DISC(0.75) WITHIN GROUP (ORDER BY order_amount) AS third_quartile_order
FROM orders_data;
```

NESTED Queries: with IN or EXISTS, you can do multiple queries and subqueries.

```sql
SELECT S_ID FROM STUDENT_COURSE 
WHERE C_ID IN (
  SELECT C_ID FROM COURSE WHERE C_NAME IN ('DSA', 'DBMS')
);

SELECT S_NAME FROM STUDENT S
WHERE EXISTS (
  SELECT 1 FROM STUDENT_COURSE SC
  WHERE S.S_ID = SC.S_ID AND SC.C_ID = 'C1'
);
```

## Cohort analysis

**Cohort analysis** groups users based on shared characteristics or events occurring at the same time — such as the date of account creation or the first investment — and examines their behavior over time. This method allows businesses to track metrics like retention, lifetime value (LTV), or churn for specific user groups. In turn, player retention can be defined through both financial actions, such as deposits, and behavioral actions, such as logins. This will be discussed in a separate article.

- Objective: Understand long-term user behavior and trends.
- Key Metrics: Retention rate, churn rate, lifetime value (LTV).
- Use Case Example: A company wants to analyze the financial behavior of customers who opened accounts in the first quarter of 2022. Using cohort analysis, they can track how these customers’ deposits grow (or decline) over subsequent months.

## Funnel analysis

**Funnel analysis** focuses on the sequence of actions users take in a specific process — for example, applying for a loan or signing up for a credit card — and identifies drop-off points. This approach helps businesses optimize user journeys to improve conversions.

- Objective: Identify bottlenecks and optimize specific user processes.
- Key Metrics: Conversion rate, drop-off rate.
- Use Case Example: A financial app wants to understand why users abandon the loan application process. By analyzing the funnel (e.g., starting an application → completing verification → submitting documents → loan approval), they can pinpoint where most users drop off.
