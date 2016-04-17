# ManagingBigData_MySQL_DukeUniv

Order of processing SQL:
![Order](week3/Order_of_processing_smaller.jpg "Order_of_processing")

General concepts behind outer joins:
<p align="center">
<img src="week3/Join_diagram.jpg" width=400/>
</p>

### Some good practices

* Avoid making assumptions about your data or your analyses. For example, rather than assume that all the values in a column are unique just because some documentation says they should be, check for yourself!

* Always look at example outputs of your queries before you strongly interpret aggregate calculations. Take extra care to do this when your queries require joins.

* When your queries require multiple layers of functions or joins, examine the output of each layer or join first before you combine them all together.

* Adopt a healthy skepticsm of all your data and results. If you see something you don't expect, make sure you explore it before interpreting it strongly or incorporating it into other analyses.


### Some point to note

* COUNT DISTINCT does NOT count NULL values
* SELECT/GROUP BY clauses roll up NULL values into one group

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

## Certificate

<p align="center">
<img src="week5/Certificate_Duke_sql.png" width=400/>
</p>
