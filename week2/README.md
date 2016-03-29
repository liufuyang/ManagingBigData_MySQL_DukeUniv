# Week2 Homework Notes

## Overall Objectives for the Teradata Practice Exercises

* Week 2: This week your aims should be to become comfortable with the Teradata front end
interface, and the small differences in MySQL and Teradata SQL syntax. You should also aim
to get a good sense of what the Dillard’s data set contains, and what kind of analysis questions
the data set could help you answer. Finish the week knowing how to retrieve, and change the
format of, data from single tables that meet specified criteria.

* Week 3: In Week 3 your aims should be to become very confident in your ability to use the
main keywords in a SQL query, and to start to appreciate how long queries take when you are
combining data from multiple tables with millions of rows. Finish the week knowing how to
summarize data, how to segment summaries of your data, and how to combine data across
multiple tables.

* Week 4: In Week 4 your aims should be to become comfortable translating analysis questions
into SQL queries, and confident that you can generate creative approaches to retrieving data in
formats that are different from the way they are stored in a database. You should also aim to
gain a deeper appreciation of how long real queries take to write and to execute. Overall, you
want to finish this week feeling like a completely independent SQL analyst. You may still have
questions, but you will know what questions to ask to succeed, and will know how to implement
any answers or help you receive.

### Specific Goals for Week 2’s Teradata exercises
Use these exercises to:
- Become comfortable with the SQL Scratchpad
- Become familiar with the Dillard’s database
- Recognize syntax differences between Teradata and MySQL
- Ensure you understand how to implement all the SQL syntax we learned this week

## Basic Teradata command

```MySQL
HELP TABLE [name of table goes here, to show columns of it]
HELP COLUMN [name of column goes here]
SHOW table [insert name of table here, to show details]
```
Terdata uses a TOP operator instead of a LIMIT operator to
restrict the length of a query output.

The following statement would select the first 10 rows of the strinfo table, ordered in ascending
alphabetical order by the city name (you would retrieve names that start with “a”)
```MySQL
SELECT TOP 10 *
FROM strinfo
ORDER BY city ASC
```
The documentation for the TOP function can be found [here](http://www.info.teradata.com/htmlpubs/db_ttu_13_10/index.html#page/SQL_Reference/B035_1146_109A/ch01.3.095.html).
The Teradata TOP function does not allow you to start the number of rows you select at a certain
row number. To select specific rows you would need to use functions such as RANK or
ROW_NUMBER within subqueries, but we will not learn about subqueries until the last week of
the course. The SQL Scratchpad does allow you to page through your output, though. In
addition, there is a function available in Teradata (but not MySQL) called [SAMPLE](http://www.teradatawiki.net/2013/10/Teradata-SAMPLE-Function.html) that allows
you to select a random sampling of the data in a table.

The following query would retrieve 10 random rows from the strinfo table:
```MySQL
SELECT *
FROM strinfo
SAMPLE 10
```
The following query would retrieve a random 10% of the rows from the strinfo table:
```MySQL
SELECT *
FROM strinfo
SAMPLE .10
```
#### Other differences
The other important differences between Teradata and MySQL you need to know about are:
- DISTINCT and LIMIT can be used in the same query statement in MySQL, but DISTINCT and TOP cannot be used together in Teradata
- MySQL accepts either double or single quotation marks around strings of text in queries,
**but Teradata will only accept single quotation marks (double quotation used behind AS???)**
- MySQL will accept the symbols “!=” and “<>” to indicate “does not equal”, **but Teradata
will only accept “<>”** (other operators, like “IN”, “BETWEEN”, and “LIKE” are the
same: [link](http://www.teradatawiki.net/2013/09/Teradata-Operators.html))

Keeping these differences in mind, and knowing that the Dillard’s database was configured so
that the names of the tables and columns are case insensitive.

## Exercise 1.
Use HELP and SHOW to confirm that the relational schema provided to us for the Dillard’s dataset shows the correct column names and primary keys for each table.
```MySQL
SHOW table trnsact;
HELP table trnsact;
```

## Exercise 2.
Look at examples of data from each of the tables. Pay particular attention to
the skuinfo table.

```MySQL
SELECT *
FROM strinfo
SAMPLE 10

SELECT *
FROM skuinfo
SAMPLE 10
```
Some things to note:
- There are two types of transactions: purchases and returns. We will need to make sure
we specify in which type we are interested, when running queries using the transaction
table.
- There are a lot of strange values in the “color”, “style”, and “size” fields of the skuinfo
table. The information recorded in these columns is not always related to the column title
(for example there are entries like "BMK/TOUR K” and “ALOE COMBO” in the color
field, even though those entries do not represent colors).
- The department descriptions seem to represent brand names. However, if you look at
entries in the skuinfo table from only one department, you will see that many brands are
in the same department.

## Exercise 3.
Examine lists of distinct values in each of the tables.

## Exercise 4.
Examine instances of transaction table where “amt” is different than “sprice”.
What did you learn about how the values in “amt”, “quantity”, and “sprice” relate to one
another?

```MySQL
SELECT *
FROM trnsact
WHERE sprice <> amt
SAMPLE 10
```
Looks like `AMT = QUANTITY * SPRICE`

## Exercise 5:
Even though the Dillard’s dataset had primary keys declared and there were
not many NULL values, there are still many strange entries that likely reflect entry errors.
To see some examples of these likely errors, examine:
* (a) rows in the trsnact table that have “0” in their orgprice column (how could the
original price be 0?),
```MySQL
SELECT *
FROM trnsact
WHERE orgprice=0
SAMPLE 10
```
* (b) rows in the skstinfo table where both the cost and retail price are listed as 0.00,
and
```MySQL
SELECT *
FROM skstinfo
WHERE cost=0 AND retail=0
SAMPLE 10
```
* (c) rows in the skstinfo table where the cost is greater than the retail price (although
occasionally retailers will sell an item at a loss for strategic reasons, it is very
unlikely that a manufacturer would provide a suggested retail price that is lower
than the cost of the item).
```MySQL
SELECT *
FROM skstinfo
WHERE cost > retail
SAMPLE 10
```

## Exercise 6.
Write your own queries that retrieve multiple columns in a precise order from
a table, and that restrict the rows retrieved from those columns using “BETWEEN”, “IN”,
and references to text strings.
```MySQL
SELECT stype, orgprice AS "Original Price", AMT, sprice, quantity AS Q
FROM trnsact
WHERE orgprice BETWEEN 10 AND 50 and quantity > 1
SAMPLE 10
```

```MySQL
HELP table skuinfo;

SELECT TOP 5 sku, color, brand
FROM skuinfo
WHERE brand='LIZ CLAI'
ORDER BY sku DESC
```
What was the largest margin (original price minus sale price) of an item sold on that earliest date?
```MySQL
SELECT TOP 5 (orgprice - sprice) AS "margin", sprice, orgprice, saledate
FROM trnsact
WHERE saledate='2004-08-01'
ORDER BY (orgprice - sprice) DESC
```
