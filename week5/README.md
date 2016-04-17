# Week5 Homework Notes

## Syntax error checklist
If your query is crashing and you can’t determine why, make sure:
- all keywords are spelled correctly
- all keywords are in the correct order
- aliases do not have keywords in them
- quotation marks are of the correct type
- semi-colons are at the end of your query, not in the middle
- all opening parentheses are matched with a closing parentheses
- there are commas between all the items in a list
- there is NOT a comma after the last item in a list
- all text strings are enclosed with the appropriate types of quotation marks
- each column name is linked with the correct table name
- all the necessary join conditions are included for each join
- aliases and title names do not contain white spaces (unless the full title is encased in
quotation marks)

## Exercise 1
How many distinct dates are there in the saledate column of the transaction
table for each month/year combination in the database?
```MySQL
SELECT COUNT(DISTINCT EXTRACT(DAY FROM saledate)) AS day_num, EXTRACT(MONTH FROM saledate) AS month_num, EXTRACT(YEAR FROM saledate) AS year_num
FROM TRNSACT
GROUP BY month_num, year_num
```

## Exercise 2
Use a CASE statement within an aggregate function to determine which sku
had the greatest total sales during the combined summer months of June, July, and August.
```MySQL
SELECT TOP 5 SUM(
  CASE WHEN (EXTRACT(MONTH from t.saledate)=6 
         OR EXTRACT(MONTH from t.saledate)=7 
         OR EXTRACT(MONTH from t.saledate)=8)
       THEN t.amt
  END
) AS total_amt_summer, 
t.sku
FROM trnsact t
GROUP BY t.sku
ORDER BY total_amt_summer DESC
```
total_amt_summer | sku
--- | --- 
1665451.38 | 4108011
1502147.00 | 3524026
1348649.00 | 5528349
1076884.47 | 3978011
927286.51 | 2783996

## Exercise 3
How many distinct dates are there in the saledate column of the transaction
table for each month/year/store combination in the database? Sort your results by the
number of days per combination in ascending order.
```MySQL
SELECT COUNT(DISTINCT EXTRACT(DAY FROM saledate)) AS day_num, 
  EXTRACT(MONTH FROM saledate) AS month_num, 
  EXTRACT(YEAR FROM saledate) AS year_num,
  store
FROM TRNSACT
GROUP BY month_num, year_num, store
ORDER BY day_num 
```

## Exercise 4
What is the average daily revenue for each store/month/year combination in
the database? Calculate this by dividing the total revenue for a group by the number of
sales days available in the transaction table for that group. 
```MySQL
SELECT TOP 50 
  SUM(t.amt) / COUNT(DISTINCT t.saledate) AS total_sale_daily, 
  t.store,
  EXTRACT(MONTH from t.saledate) AS month_num,
  EXTRACT(YEAR from t.saledate) AS year_num,
  COUNT(DISTINCT t.saledate) AS num_of_days,
  CASE 
    WHEN month_num=8 AND year_num=2005 THEN 1
    ELSE 0
  END AS to_remove
FROM trnsact t
WHERE (to_remove<>1)
GROUP BY t.store, month_num, year_num
HAVING num_of_days >= 20
ORDER BY total_sale_daily DESC
```
The same but with a subquery
```MySQL
SELECT TOP 50 
  SUM(t.amt) / COUNT(DISTINCT t.saledate) AS avg_sale_daily, 
  t.store,
  EXTRACT(MONTH from t.saledate) AS month_num,
  EXTRACT(YEAR from t.saledate) AS year_num,
  TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate))
  AS year_month,
  TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate)) || '-'||TRIM(t.store) 
  AS year_month_store
FROM 
  trnsact
  AS t
WHERE (
    TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate)) || '-'||TRIM(t.store)
    IN 
    (SELECT 
      TRIM(EXTRACT(YEAR from t_s.saledate)) ||'-'|| TRIM(EXTRACT(MONTH from t_s.saledate)) || '-'||TRIM(t_s.store) AS year_month_store2
     FROM trnsact t_s
     GROUP BY year_month_store2, t_s.store
     HAVING COUNT(DISTINCT t_s.saledate) >= 20
    ) 
    AND (
      TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate))
      NOT='2005-8'
    )
)
GROUP BY t.store, month_num, year_num
ORDER BY avg_sale_daily DESC
```

And now another one with only purchases
```MySQL
SELECT TOP 50 
  SUM(t.amt) / COUNT(DISTINCT t.saledate) AS avg_sale_daily, 
  t.store,
  EXTRACT(MONTH from t.saledate) AS month_num,
  EXTRACT(YEAR from t.saledate) AS year_num,
  TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate))
  AS year_month,
  TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate)) || '-'||TRIM(t.store) 
  AS year_month_store
FROM 
  trnsact
  AS t
WHERE (
    TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate)) || '-'||TRIM(t.store)
    IN 
    (SELECT 
      TRIM(EXTRACT(YEAR from t_s.saledate)) ||'-'|| TRIM(EXTRACT(MONTH from t_s.saledate)) || '-'||TRIM(t_s.store) AS year_month_store2
     FROM trnsact t_s
     GROUP BY year_month_store2, t_s.store
     HAVING COUNT(DISTINCT t_s.saledate) >= 20
    ) 
    AND (
      TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate))
      NOT='2005-8'
    )
    AND t.stype='P'
)
GROUP BY t.store, month_num, year_num
ORDER BY avg_sale_daily DESC
```

Save your final queries that remove “bad data” for use in subsequent exercises. 

From now on (and in the graded quiz), when I ask for average daily revenue:
* Only examine purchases (not returns)
* Exclude all stores with less than 20 days of data
* Exclude all data from August, 2005

## Exercise 5
What is the average daily revenue brought in by Dillard’s stores in areas of
high, medium, or low levels of high school education?

Define areas of “low” education as those that have high school graduation rates between
50-60%, areas of “medium” education as those that have high school graduation rates between
60-70%, and areas of “high” education as those that have high school graduation rates of above
70%.

```MySQL
SELECT TOP 50 
  SUM(t.amt) / COUNT(DISTINCT t.saledate) AS total_sale_daily, 
  sm.level_high_school
FROM 
  trnsact AS t
  JOIN
  (
    SELECT store, CASE  
    WHEN msa_high>=50.0 AND msa_high<60.0 THEN 'low'
    WHEN msa_high>=60.0 AND msa_high<70.0 THEN 'medium'
    WHEN msa_high>=70.0 THEN 'high'
    END AS level_high_school
    FROM store_msa
  )
    AS sm
    ON sm.store=t.store
WHERE (
    TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate)) || '-'||TRIM(t.store)
    IN 
    (SELECT 
      TRIM(EXTRACT(YEAR from t_s.saledate)) ||'-'|| TRIM(EXTRACT(MONTH from t_s.saledate)) || '-'||TRIM(t_s.store) AS year_month_store2
     FROM trnsact t_s
     GROUP BY year_month_store2, t_s.store
     HAVING COUNT(DISTINCT t_s.saledate) >= 20
    ) 
    AND (
      TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate))
      NOT='2005-8'
    )
  AND t.stype='P'
)
GROUP BY sm.level_high_school
ORDER BY total_amt_summer DESC
```
With stores grouped as well:
```MySQL
SELECT TOP 50 
  SUM(t.amt) / COUNT(DISTINCT t.saledate) AS total_amt_summer, 
  t.store,
  sm.level_high_school
FROM 
  trnsact AS t
  JOIN
  (
    SELECT store, CASE  
    WHEN msa_high>=50.0 AND msa_high<60.0 THEN 'low'
    WHEN msa_high>=60.0 AND msa_high<70.0 THEN 'medium'
    WHEN msa_high>=70.0 THEN 'high'
    END AS level_high_school
    FROM store_msa
  )
    AS sm
    ON sm.store=t.store
WHERE (
    TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate)) || '-'||TRIM(t.store)
    IN 
    (SELECT 
      TRIM(EXTRACT(YEAR from t_s.saledate)) ||'-'|| TRIM(EXTRACT(MONTH from t_s.saledate)) || '-'||TRIM(t_s.store) AS year_month_store2
     FROM trnsact t_s
     GROUP BY year_month_store2, t_s.store
     HAVING COUNT(DISTINCT t_s.saledate) >= 20
    ) 
    AND (
      TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate))
      NOT='2005-8'
    )
  AND t.stype='P'
)
GROUP BY t.store, sm.level_high_school
ORDER BY total_amt_summer DESC
```

## Exercise 6
Compare the average daily revenues of the stores with the highest median
msa_income and the lowest median msa_income.

In what city and state were these stores,
and which store had a higher average daily revenue? 
```MySQL
SELECT TOP 50 
  SUM(t.amt) / COUNT(DISTINCT t.saledate) AS total_amt_summer, 
  t.store,
  sm.level_high_school, sm.msa_income,
  strinfo.city, strinfo.state
FROM 
  trnsact AS t
  JOIN
  (
    SELECT sm_fi.store, CASE  
    WHEN sm_fi.msa_high>=50.0 AND sm_fi.msa_high<60.0 THEN 'low'
    WHEN sm_fi.msa_high>=60.0 AND sm_fi.msa_high<70.0 THEN 'medium'
    WHEN sm_fi.msa_high>=70.0 THEN 'high'
    END AS level_high_school,
    sm_fi.msa_income
    FROM (
      SELECT TOP 3 store, msa_high, msa_income
      FROM store_msa
      ORDER BY msa_income DESC
    ) AS sm_fi
  )
    AS sm
    ON sm.store=t.store
    JOIN strinfo
    ON strinfo.store=sm.store
WHERE (
    TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate)) || '-'||TRIM(t.store)
    IN 
    (SELECT 
      TRIM(EXTRACT(YEAR from t_s.saledate)) ||'-'|| TRIM(EXTRACT(MONTH from t_s.saledate)) || '-'||TRIM(t_s.store) AS year_month_store2
     FROM trnsact t_s
     GROUP BY year_month_store2, t_s.store
     HAVING COUNT(DISTINCT t_s.saledate) >= 20
    ) 
    AND (
      TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate))
      NOT='2005-8'
    )
  AND t.stype='P'
)
GROUP BY t.store, sm.level_high_school, sm.msa_income, strinfo.city, strinfo.state
ORDER BY total_amt_summer DESC
```
Daily AVG| store | school level | msa_income |city|state
---|---|---|---|---|---
17884.08 |3902 |high |56099 |SPANISH FORT | AL
14644.10 |9609 |high |45944 |LONGMONT |CO
10072.60 |3403 |high |46140 |SALINA |KS
56601.99 |2707 |low |16022 |MCALLEN | TX
34590.64 |2907 |low |17374 |BROWNSVILLE | TX
16479.60 |2807 |low |17374 |HARLINGEN | TX

## Exercise 7
What is the brand of the sku with the greatest standard deviation in sprice?
Only examine skus that have been part of over 100 transactions.
```MySQL
SELECT TOP 3 sku, STDDEV_SAMP(ALL sprice) AS std_sprice
FROM trnsact
WHERE sku IN (
  SELECT sku
  FROM trnsact
  GROUP BY sku
  HAVING COUNT(*)>=100 
)
GROUP BY sku
ORDER BY std_sprice DESC
```
Resulting 178.6 as the highest std_sprice for sku 3733090.

## Exercise 8
Examine all the transactions for the sku with the greatest standard deviation in sprice, but only consider skus that are part of more than 100 transactions.
```MySQL
SELECT sprice, saledate
FROM trnsact
WHERE sku=(
  SELECT top1.sku
  FROM (
    SELECT TOP 1 sku
    FROM trnsact
    WHERE sku IN (
        SELECT sku
        FROM trnsact
        GROUP BY sku
        HAVING COUNT(*)>=100 
      )
    GROUP BY sku
    ORDER BY STDDEV_SAMP(ALL sprice) DESC
  ) AS top1
)
ORDER BY sprice DESC
```

Or simply using the results above:
```MySQL
SELECT sprice, saledate
FROM trnsact
WHERE sku=3733090
ORDER BY sprice DESC
```

Resulting in most of the price is 5 to 6 dollars but there are two records marked as 5005.

## Exercise 9
What was the average daily revenue Dillard’s brought in during each month of the year?
```
SELECT
  SUM(t.amt) / COUNT(DISTINCT t.saledate) AS avg_daily_sale, 
  EXTRACT(MONTH from t.saledate) AS month_num
FROM 
  trnsact
  AS t
WHERE (
    TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate)) || '-'||TRIM(t.store)
    IN 
    (SELECT 
      TRIM(EXTRACT(YEAR from t_s.saledate)) ||'-'|| TRIM(EXTRACT(MONTH from t_s.saledate)) || '-'||TRIM(t_s.store) AS year_month_store2
     FROM trnsact t_s
     GROUP BY year_month_store2, t_s.store
     HAVING COUNT(DISTINCT t_s.saledate) >= 20
    ) 
    AND (
      TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate))
      NOT='2005-8'
    )
  AND t.stype='P'
)
GROUP BY month_num
ORDER BY avg_daily_sale DESC
```
Best months: 12, 2, 7, 4;
Worst months: 9, 8, 1, 10

## Exercise 10
Which department, in which city and state of what store, had the greatest % increase in average daily sales revenue from November to December? 
```MySQL
SELECT TOP 10
  t.store, sm.city, sm.state, dp.deptdesc,
  SUM(
    CASE EXTRACT(MONTH from t.saledate)
    WHEN 11 THEN t.amt
    END
  ) AS daily_month_11,
  SUM(
    CASE EXTRACT(MONTH from t.saledate)
    WHEN 12 THEN t.amt
    END
  )  AS daily_month_12,

  COUNT( DISTINCT
    CASE EXTRACT(MONTH from t.saledate)
    WHEN 11 THEN t.saledate
    END
  ) AS day_num_11,
  COUNT( DISTINCT
    CASE EXTRACT(MONTH from t.saledate)
    WHEN 12 THEN t.saledate
    END
  ) AS day_num_12,
  
  daily_month_11/day_num_11 AS avg_dsale_m11,
  daily_month_12/day_num_12 AS avg_dsale_m12,
  (avg_dsale_m12-avg_dsale_m11)/avg_dsale_m11*100 AS increase
FROM 
  trnsact
  AS t 
  JOIN store_msa sm ON sm.store=t.store
  JOIN skuinfo sk ON sk.sku=t.sku
  JOIN deptinfo dp ON dp.dept=sk.dept
WHERE (
    TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate)) || '-'||TRIM(t.store)
    IN 
    (SELECT 
      TRIM(EXTRACT(YEAR from t_s.saledate)) ||'-'|| TRIM(EXTRACT(MONTH from t_s.saledate)) || '-'||TRIM(t_s.store) AS year_month_store2
     FROM trnsact t_s
     GROUP BY year_month_store2, t_s.store
     HAVING COUNT(DISTINCT t_s.saledate) >= 20
    ) 
    AND (
      TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate))
      NOT='2005-8'
    ) 
  AND t.stype='P'
)
GROUP BY t.store, sm.city, sm.state, dp.deptdesc
ORDER BY increase DESC
```

## Exercise 11
What is the city and state of the store that had the greatest decrease in average daily revenue from August to September?
```MySQL
SELECT TOP 10
  t.store, sm.city, sm.state,
  SUM(
    CASE EXTRACT(MONTH from t.saledate)
    WHEN 8 THEN t.amt
    END
  ) AS daily_month_8,
  SUM(
    CASE EXTRACT(MONTH from t.saledate)
    WHEN 9 THEN t.amt
    END
  )  AS daily_month_9,

  COUNT( DISTINCT
    CASE EXTRACT(MONTH from t.saledate)
    WHEN 8 THEN t.saledate
    END
  ) AS day_num_8,
  COUNT( DISTINCT
    CASE EXTRACT(MONTH from t.saledate)
    WHEN 9 THEN t.saledate
    END
  ) AS day_num_9,
  
  daily_month_8/day_num_8 AS avg_dsale_m8,
  daily_month_9/day_num_9 AS avg_dsale_m9,
  (avg_dsale_m8-avg_dsale_m9)/avg_dsale_m8*100 AS decrease
FROM 
  trnsact
  AS t 
  JOIN store_msa sm ON sm.store=t.store
  JOIN skuinfo sk ON sk.sku=t.sku
WHERE (
    TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate)) || '-'||TRIM(t.store)
    IN 
    (SELECT 
      TRIM(EXTRACT(YEAR from t_s.saledate)) ||'-'|| TRIM(EXTRACT(MONTH from t_s.saledate)) || '-'||TRIM(t_s.store) AS year_month_store2
     FROM trnsact t_s
     GROUP BY year_month_store2, t_s.store
     HAVING COUNT(DISTINCT t_s.saledate) >= 20
    ) 
    AND (
      TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate))
      NOT='2005-8'
    ) 
  AND t.stype='P'
)
GROUP BY t.store, sm.city, sm.state
HAVING daily_month_8>0 AND daily_month_9>0
ORDER BY decrease
```

## Exercise 12: 
Determine the month of maximum total revenue for each store. Count the number of stores whose month of maximum total revenue was in each of the twelve
months. Then determine the month of maximum average daily revenue. Count the number of stores whose month of maximum average daily revenue was in each of the twelve months. How do they compare?
```MySQL
SELECT TOP 50 
  SUM(t.amt) / COUNT(DISTINCT t.saledate) AS avg_daily_sale, 
  SUM(t.amt) AS total_month_sale,
  ROW_NUMBER() OVER (ORDER BY total_month_sale DESC) AS total_month_rownum,
  ROW_NUMBER() OVER (ORDER BY avg_daily_sale DESC) AS avg_daily_rownum,
  t.store,
  EXTRACT(MONTH from t.saledate) AS month_num,
  EXTRACT(YEAR from t.saledate) AS year_num,
  TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate)) AS year_month_num
FROM 
  trnsact
  AS t
WHERE (
    TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate)) || '-'||TRIM(t.store)
    IN 
    (SELECT 
      TRIM(EXTRACT(YEAR from t_s.saledate)) ||'-'|| TRIM(EXTRACT(MONTH from t_s.saledate)) || '-'||TRIM(t_s.store) AS year_month_store2
     FROM trnsact t_s
     GROUP BY year_month_store2, t_s.store
     HAVING COUNT(DISTINCT t_s.saledate) >= 20
    ) 
    AND (
      TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate))
      NOT='2005-8'
    ) 
  AND t.stype='P'
)
GROUP BY t.store, month_num, year_num
ORDER BY total_month_sale DESC
```
For average daily revenue:
```MySQL
SELECT top_store_month.month_num, COUNT(top_store_month.store) AS num_of_stores

FROM (SELECT
  SUM(t.amt) / COUNT(DISTINCT t.saledate) AS avg_daily_sale, 
  ROW_NUMBER() OVER (PARTITION BY t.store ORDER BY avg_daily_sale DESC) AS avg_daily_rownum,
  t.store,
  EXTRACT(MONTH from t.saledate) AS month_num
FROM 
  trnsact
  AS t
WHERE (
    TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate)) || '-'||TRIM(t.store)
    IN 
    (SELECT 
      TRIM(EXTRACT(YEAR from t_s.saledate)) ||'-'|| TRIM(EXTRACT(MONTH from t_s.saledate)) || '-'||TRIM(t_s.store) AS year_month_store2
     FROM trnsact t_s
     GROUP BY year_month_store2, t_s.store
     HAVING COUNT(DISTINCT t_s.saledate) >= 20
    ) 
    AND (
      TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate))
      NOT='2005-8'
    ) 
  AND t.stype='P'
)
GROUP BY t.store, month_num
QUALIFY avg_daily_rownum=1) 

AS top_store_month
GROUP BY top_store_month.month_num
```
With result for December: 316 stores have highest rank; March: 6 stores have highest rank. July next having 2 stores.

For total month revenue:
```MySQL
SELECT top_store_month.month_num, COUNT(top_store_month.store) AS num_of_stores

FROM (SELECT
  SUM(t.amt) AS total_month_sale, 
  ROW_NUMBER() OVER (PARTITION BY t.store ORDER BY total_month_sale DESC) AS avg_daily_rownum,
  t.store,
  EXTRACT(MONTH from t.saledate) AS month_num
FROM 
  trnsact
  AS t
WHERE (
    TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate)) || '-'||TRIM(t.store)
    IN 
    (SELECT 
      TRIM(EXTRACT(YEAR from t_s.saledate)) ||'-'|| TRIM(EXTRACT(MONTH from t_s.saledate)) || '-'||TRIM(t_s.store) AS year_month_store2
     FROM trnsact t_s
     GROUP BY year_month_store2, t_s.store
     HAVING COUNT(DISTINCT t_s.saledate) >= 20
    ) 
    AND (
      TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate))
      NOT='2005-8'
    ) 
  AND t.stype='P'
)
GROUP BY t.store, month_num
QUALIFY avg_daily_rownum=1) 

AS top_store_month
GROUP BY top_store_month.month_num
```
With result for December: 321 stores have highest rank; March: 4 stores have highest rank. July having 4.

## A note about missing data
If you encounter a data set in your professional life that is missing as much data as we are
missing in our Dillard's data set, I would recommend taking steps we do not have an opportunity to take here. 

In particular, do everything you can to find out why those data are missing, and to
fill in missing data with other sources of information. You could also try to take advantage of other data sets to help you make statistical guesses about what the sales would likely have been, or to standardize your calculations more accurately (you could calculate revenue on an hourly basis if you knew how long each store was open on each day, or on a sqare-footage basis if you knew how large each store was, for example). 

The most important things to take away from the
strategies we used in these exercises are that: 

1. real data sets are messy and it is your job as an analyst to find that mess so that you can take it into account, 
2. you can design your SQL
queries to accommodate data anomalies, and 
3. avoid using averages of averages to summarize
the performance of a group.

# Quiz 
### 1. 
#### How many distinct skus have the brand “Polo fas”, and are either size “XXL” or “black” in color?
```MySQL
SELECT COUNT(DISTINCT
  CASE 
    WHEN brand='Polo fas' AND (size='XXL' OR color='black')
    THEN sku
  END
)
FROM skuinfo
```
13623

### 2. 
#### There was one store in the database which had only 11 days in one of its months (in other words, that store/month/year combination only contained 11 days of transaction data). In what city and state was this store located?

```MySQL
SELECT 
      TRIM(EXTRACT(YEAR from t_s.saledate)) ||'-'|| TRIM(EXTRACT(MONTH from t_s.saledate) )AS year_month_num2
 ,sm.city, sm.state
FROM trnsact t_s JOIN store_msa sm ON sm.store=t_s.store
GROUP BY year_month_num2, t_s.store, sm.city, sm.state
HAVING COUNT(DISTINCT t_s.saledate) = 11
```
Atlanta GA

### 3. 
#### Which sku number had the greatest increase in total sales revenue from November to December?
```MySQL
SELECT TOP 10
  sk.sku,
  SUM(
    CASE EXTRACT(MONTH from t.saledate)
    WHEN 11 THEN t.amt
    END
  ) AS daily_month_11,
  SUM(
    CASE EXTRACT(MONTH from t.saledate)
    WHEN 12 THEN t.amt
    END
  )  AS daily_month_12,

  (daily_month_12-daily_month_11) AS increase
FROM 
  trnsact
  AS t 
  JOIN store_msa sm ON sm.store=t.store
  JOIN skuinfo sk ON sk.sku=t.sku
  JOIN deptinfo dp ON dp.dept=sk.dept
WHERE (
    TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate)) || '-'||TRIM(t.store)
    IN 
    (SELECT 
      TRIM(EXTRACT(YEAR from t_s.saledate)) ||'-'|| TRIM(EXTRACT(MONTH from t_s.saledate)) || '-'||TRIM(t_s.store) AS year_month_store2
     FROM trnsact t_s
     GROUP BY year_month_store2, t_s.store
     HAVING COUNT(DISTINCT t_s.saledate) >= 20
    ) 
    AND (
      TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate))
      NOT='2005-8'
    ) 
  AND t.stype='P'
)
GROUP BY sk.sku
ORDER BY increase DESC
```
3949538

### 4. 
#### What vendor has the greatest number of distinct skus in the transaction table that do not exist in the skstinfo table? (Remember that vendors are listed as distinct numbers in our data set).
```MySQL
SELECT TOP 10 sk.vendor, COUNT(Distinct t.sku) AS sku_num
FROM trnsact t JOIN skuinfo sk ON sk.sku=t.sku
WHERE NOT EXISTS (
  SELECT * FROM skstinfo 
  WHERE skstinfo.sku=t.sku
)
GROUP BY sk.vendor
ORDER BY sku_num DESC
```
5715232

### 5. 
#### What is the brand of the sku with the greatest standard deviation in sprice? Only examine skus which have been part of over 100 transactions.
```MySQL
SELECT TOP 10 sk.sku, STDDEV_SAMP(ALL t.sprice) AS std_sprice, sk.brand
FROM trnsact t JOIN skuinfo sk ON sk.sku=t.sku
WHERE t.sku IN (
  SELECT sku
  FROM trnsact
  GROUP BY sku
  HAVING COUNT(*)>100 
)
GROUP BY sk.sku, sk.brand
ORDER BY std_sprice DESC
```
Hart Sch

### 6.
#### What is the city and state of the store which had the greatest increase in average daily revenue (as I define it in Teradata Week 5 Exercise Guide) from November to December?
```MySQL
SELECT TOP 10
  t.store, sm.city, sm.state,
  SUM(
    CASE EXTRACT(MONTH from t.saledate)
    WHEN 11 THEN t.amt
    END
  ) AS daily_month_11,
  SUM(
    CASE EXTRACT(MONTH from t.saledate)
    WHEN 12 THEN t.amt
    END
  )  AS daily_month_12,

  COUNT( DISTINCT
    CASE EXTRACT(MONTH from t.saledate)
    WHEN 11 THEN t.saledate
    END
  ) AS day_num_11,
  COUNT( DISTINCT
    CASE EXTRACT(MONTH from t.saledate)
    WHEN 12 THEN t.saledate
    END
  ) AS day_num_12,
  
  daily_month_11/day_num_11 AS avg_dsale_m11,
  daily_month_12/day_num_12 AS avg_dsale_m12,
  (avg_dsale_m12-avg_dsale_m11) AS increase
FROM 
  trnsact
  AS t 
  JOIN store_msa sm ON sm.store=t.store
  JOIN skuinfo sk ON sk.sku=t.sku
WHERE (
    TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate)) || '-'||TRIM(t.store)
    IN 
    (SELECT 
      TRIM(EXTRACT(YEAR from t_s.saledate)) ||'-'|| TRIM(EXTRACT(MONTH from t_s.saledate)) || '-'||TRIM(t_s.store) AS year_month_store2
     FROM trnsact t_s
     GROUP BY year_month_store2, t_s.store
     HAVING COUNT(DISTINCT t_s.saledate) >= 20
    ) 
    AND (
      TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate))
      NOT='2005-8'
    ) 
  AND t.stype='P'
)
GROUP BY t.store, sm.city, sm.state
ORDER BY increase DESC
```

### 7.
#### Compare the average daily revenue (as I define it in Teradata Week 5 Exercise Guide) of the store with the highest msa_income and the store with the lowest median msa_income (according to the msa_income field). In what city and state were these two stores, and which store had a higher average daily revenue?
See Excercise 6 above.

### 8.
#### Divide the msa_income groups up so that msa_incomes between 1 and 20,000 are labeled 'low', msa_incomes between 20,001 and 30,000 are labeled 'med-low', msa_incomes between 30,001 and 40,000 are labeled 'med-high', and msa_incomes between 40,001 and 60,000 are labeled 'high'. Which of these groups has the highest average daily revenue (as I define it in Teradata Week 5 Exercise Guide) per store?

```MySQL
SELECT TOP 50 
  SUM(t.amt) / COUNT(DISTINCT t.saledate) / COUNT(DISTINCT t.store) AS total_amt_summer, 
  sm.level_msa_income
FROM 
  trnsact AS t
  JOIN
  (
    SELECT sm_fi.store, CASE  
       WHEN sm_fi.msa_income>=1.0 AND sm_fi.msa_income<20000.0 THEN 'low'
       WHEN sm_fi.msa_income>=20000 AND sm_fi.msa_income<30000.0 THEN 'med-low'
       WHEN sm_fi.msa_income>=30000 AND sm_fi.msa_income<40000.0 THEN 'med-high'
       WHEN sm_fi.msa_income>=40000 AND sm_fi.msa_income<60000.0 THEN 'high'
       END AS level_msa_income
    FROM store_msa sm_fi
  )
    AS sm
    ON sm.store=t.store
    JOIN strinfo
    ON strinfo.store=sm.store
WHERE (
    TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate)) || '-'||TRIM(t.store)
    IN 
    (SELECT 
      TRIM(EXTRACT(YEAR from t_s.saledate)) ||'-'|| TRIM(EXTRACT(MONTH from t_s.saledate)) || '-'||TRIM(t_s.store) AS year_month_store2
     FROM trnsact t_s
     GROUP BY year_month_store2, t_s.store
     HAVING COUNT(DISTINCT t_s.saledate) >= 20
    ) 
    AND (
      TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate))
      NOT='2005-8'
    )
  AND t.stype='P'
)
GROUP BY  sm.level_msa_income
ORDER BY total_amt_summer DESC
```
low...

### 9. 
#### Divide stores up so that stores with msa populations between 1 and 100,000 are labeled 'very small', stores with msa populations between 100,001 and 200,000 are labeled 'small', stores with msa populations between 200,001 and 500,000 are labeled 'med_small', stores with msa populations between 500,001 and 1,000,000 are labeled 'med_large', stores with msa populations between 1,000,001 and 5,000,000 are labeled “large”, and stores with msa_population greater than 5,000,000 are labeled “very large”. What is the average daily revenue (as I define it in Teradata Week 5 Exercise Guide) for a store in a “very large” population msa?

```MySQL
SELECT TOP 50 
  SUM(t.amt) / COUNT(DISTINCT t.saledate) / COUNT(DISTINCT t.store) AS total_amt_summer, 
  sm.level_msa_pop
FROM 
  trnsact AS t
  JOIN
  (
    SELECT sm_fi.store, CASE  
       WHEN sm_fi.msa_pop>=1.0000 AND sm_fi.msa_pop<100001.0 THEN 'very small'
       WHEN sm_fi.msa_pop>=100001 AND sm_fi.msa_pop<200001.0 THEN 'small'
       WHEN sm_fi.msa_pop>=200001 AND sm_fi.msa_pop<500001.0 THEN 'med-small'
       WHEN sm_fi.msa_pop>=500001 AND sm_fi.msa_pop<1000001.0 THEN 'med-large'
       WHEN sm_fi.msa_pop>=1000001 AND sm_fi.msa_pop<5000000.0 THEN 'large'
       WHEN sm_fi.msa_pop>=5000000 THEN 'very large'
       END AS level_msa_pop
    FROM store_msa sm_fi
  )
    AS sm
    ON sm.store=t.store
    JOIN strinfo
    ON strinfo.store=sm.store
WHERE (
    TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate)) || '-'||TRIM(t.store)
    IN 
    (SELECT 
      TRIM(EXTRACT(YEAR from t_s.saledate)) ||'-'|| TRIM(EXTRACT(MONTH from t_s.saledate)) || '-'||TRIM(t_s.store) AS year_month_store2
     FROM trnsact t_s
     GROUP BY year_month_store2
     HAVING COUNT(DISTINCT t_s.saledate) >= 20
    ) 
    AND (
      TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate))
      NOT='2005-8'
    )
  AND t.stype='P'
)
GROUP BY  sm.level_msa_pop
ORDER BY total_amt_summer DESC
```
24948 for very large.


### 10. 
#### Which department in which store had the greatest percent increase in average daily sales revenue from November to December, and what city and state was that store located in? Only examine departments whose total sales were at least $1,000 in both November and December.
```MySQL
SELECT TOP 10
  t.store, sm.city, sm.state, dp.deptdesc,
  SUM(
    CASE EXTRACT(MONTH from t.saledate)
    WHEN 11 THEN t.amt
    END
  ) AS daily_month_11,
  SUM(
    CASE EXTRACT(MONTH from t.saledate)
    WHEN 12 THEN t.amt
    END
  )  AS daily_month_12,

  COUNT( DISTINCT
    CASE EXTRACT(MONTH from t.saledate)
    WHEN 11 THEN t.saledate
    END
  ) AS day_num_11,
  COUNT( DISTINCT
    CASE EXTRACT(MONTH from t.saledate)
    WHEN 12 THEN t.saledate
    END
  ) AS day_num_12,
  
  daily_month_11/day_num_11 AS avg_dsale_m11,
  daily_month_12/day_num_12 AS avg_dsale_m12,
  (avg_dsale_m12-avg_dsale_m11)/avg_dsale_m11*100 AS increase
FROM 
  trnsact
  AS t 
  JOIN store_msa sm ON sm.store=t.store
  JOIN skuinfo sk ON sk.sku=t.sku
  JOIN deptinfo dp ON dp.dept=sk.dept
WHERE (
    TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate)) || '-'||TRIM(t.store)
    IN 
    (SELECT 
      TRIM(EXTRACT(YEAR from t_s.saledate)) ||'-'|| TRIM(EXTRACT(MONTH from t_s.saledate)) || '-'||TRIM(t_s.store) AS year_month_store2
     FROM trnsact t_s
     GROUP BY year_month_store2, t_s.store
     HAVING COUNT(DISTINCT t_s.saledate) >= 20
    ) 
    AND (
      TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate))
      NOT='2005-8'
    ) 
  AND t.stype='P'
)
GROUP BY t.store, sm.city, sm.state, dp.deptdesc
HAVING daily_month_11>=1000
ORDER BY increase DESC
```
Louisvl dept in Salina, KS

### 11. 
#### Which department within a particular store had the greatest decrease in average daily sales revenue from August to September, and in what city and state was that store located?
Refer to Excercise 11 above
```MySQL
SELECT TOP 10
  t.store, sm.city, sm.state,  dp.deptdesc,
  SUM(
    CASE EXTRACT(MONTH from t.saledate)
    WHEN 8 THEN t.amt
    END
  ) AS daily_month_8,
  SUM(
    CASE EXTRACT(MONTH from t.saledate)
    WHEN 9 THEN t.amt
    END
  )  AS daily_month_9,

  COUNT( DISTINCT
    CASE EXTRACT(MONTH from t.saledate)
    WHEN 8 THEN t.saledate
    END
  ) AS day_num_8,
  COUNT( DISTINCT
    CASE EXTRACT(MONTH from t.saledate)
    WHEN 9 THEN t.saledate
    END
  ) AS day_num_9,
  
  daily_month_8/day_num_8 AS avg_dsale_m8,
  daily_month_9/day_num_9 AS avg_dsale_m9,
  (avg_dsale_m8-avg_dsale_m9) AS decrease
FROM 
  trnsact
  AS t 
  JOIN store_msa sm ON sm.store=t.store
  JOIN skuinfo sk ON sk.sku=t.sku
  JOIN deptinfo dp ON dp.dept=sk.dept
WHERE (
    TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate)) || '-'||TRIM(t.store)
    IN 
    (SELECT 
      TRIM(EXTRACT(YEAR from t_s.saledate)) ||'-'|| TRIM(EXTRACT(MONTH from t_s.saledate)) || '-'||TRIM(t_s.store) AS year_month_store2
     FROM trnsact t_s
     GROUP BY year_month_store2, t_s.store
     HAVING COUNT(DISTINCT t_s.saledate) >= 20
    ) 
    AND (
      TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate))
      NOT='2005-8'
    ) 
  AND t.stype='P'
)
GROUP BY t.store, sm.city, sm.state, dp.deptdesc
HAVING daily_month_8>0 AND daily_month_9>0
ORDER BY decrease DESC
```
Clinique department, Louisville, KY

### 12. ?
#### Identify the department within a particular store that had the greatest decrease innumber of items sold from August to September. How many fewer items did that department sell in September compared to August, and in what city and state was that store located?
```MySQL
SELECT TOP 10
  t.store, sm.city, sm.state,  dp.deptdesc,
  SUM(
    CASE EXTRACT(MONTH from t.saledate)
    WHEN 8 THEN t.quantity
    END
  ) AS daily_month_8,
  SUM(
    CASE EXTRACT(MONTH from t.saledate)
    WHEN 9 THEN t.quantity
    END
  )  AS daily_month_9,
  
  daily_month_8 - daily_month_9 AS decrease
FROM 
  trnsact
  AS t 
  JOIN store_msa sm ON sm.store=t.store
  JOIN skuinfo sk ON sk.sku=t.sku
  JOIN deptinfo dp ON dp.dept=sk.dept
WHERE (
    TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate)) || '-'||TRIM(t.store)
    IN 
    (SELECT 
      TRIM(EXTRACT(YEAR from t_s.saledate)) ||'-'|| TRIM(EXTRACT(MONTH from t_s.saledate)) || '-'||TRIM(t_s.store) AS year_month_store2
     FROM trnsact t_s
     GROUP BY year_month_store2, t_s.store
     HAVING COUNT(DISTINCT t_s.saledate) >= 20
    ) 
    AND (
      TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate))
      NOT='2005-8'
    ) 
  AND t.stype='P'
)
GROUP BY t.store, sm.city, sm.state, dp.deptdesc
HAVING daily_month_8>0 AND daily_month_9>0
ORDER BY decrease DESC
```
The Clinique department in Louisville, KY sold 13491 fewer items


### 13.
#### For each store, determine the month with the minimum average daily revenue (as I define it in Teradata Week 5 Exercise Guide) . For each of the twelve months of the year, count how many stores' minimum average daily revenue was in that month. During which month(s) did over 100 stores have their minimum average daily revenue?
```MySQL
SELECT top_store_month.month_num, COUNT(top_store_month.store) AS num_of_stores

FROM (SELECT
  SUM(t.amt) / COUNT(DISTINCT t.saledate) AS avg_daily_sale, 
  ROW_NUMBER() OVER (PARTITION BY t.store ORDER BY avg_daily_sale ASC) AS avg_daily_rownum,
  t.store,
  EXTRACT(MONTH from t.saledate) AS month_num
FROM 
  trnsact
  AS t
WHERE (
    TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate)) || '-'||TRIM(t.store)
    IN 
    (SELECT 
      TRIM(EXTRACT(YEAR from t_s.saledate)) ||'-'|| TRIM(EXTRACT(MONTH from t_s.saledate)) || '-'||TRIM(t_s.store) AS year_month_store2
     FROM trnsact t_s
     GROUP BY year_month_store2, t_s.store
     HAVING COUNT(DISTINCT t_s.saledate) >= 20
    ) 
    AND (
      TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate))
      NOT='2005-8'
    ) 
  AND t.stype='P'
)
GROUP BY t.store, month_num
QUALIFY avg_daily_rownum=1) 

AS top_store_month
GROUP BY top_store_month.month_num
```
August Only

### 14.
#### Write a query that determines the month in which each store had its maximum number of sku units returned. During which month did the greatest number of stores have their maximum number of sku units returned?
```MySQL
SELECT top_store_month.month_num, COUNT(top_store_month.store) AS num_of_stores

FROM (SELECT
  SUM(t.quantity) AS total_return, 
  ROW_NUMBER() OVER (PARTITION BY t.store ORDER BY total_return DESC) AS total_return_rownum,
  t.store,
  EXTRACT(MONTH from t.saledate) AS month_num
FROM 
  trnsact
  AS t
WHERE (
    TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate)) || '-'||TRIM(t.store)
    IN 
    (SELECT 
      TRIM(EXTRACT(YEAR from t_s.saledate)) ||'-'|| TRIM(EXTRACT(MONTH from t_s.saledate)) || '-'||TRIM(t_s.store) AS year_month_store2
     FROM trnsact t_s
     GROUP BY year_month_store2, t_s.store
     HAVING COUNT(DISTINCT t_s.saledate) >= 20
    ) 
    AND (
      TRIM(EXTRACT(YEAR from t.saledate))||'-'|| TRIM(EXTRACT(MONTH from t.saledate))
      NOT='2005-8'
    ) 
  AND t.stype='R'
)
GROUP BY t.store, month_num
QUALIFY total_return_rownum=1) 

AS top_store_month
GROUP BY top_store_month.month_num
```
Dec has the most returns.

## Certificate

<p align="center">
<img src="Certificate_Duke_sql.png" width=400/>
</p>
