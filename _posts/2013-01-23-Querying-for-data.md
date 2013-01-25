---
layout: post
title: Querying for data
---


Full syntax


    SELECT
    [ALL | DISTINCT | DISTINCTROW ]
      [HIGH_PRIORITY]
      [STRAIGHT_JOIN]
      [SQL_SMALL_RESULT] [SQL_BIG_RESULT] [SQL_BUFFER_RESULT]
      [SQL_CACHE | SQL_NO_CACHE] [SQL_CALC_FOUND_ROWS]
    select_expr [, select_expr ...]
    [FROM table_references
    [WHERE where_condition]
    [GROUP BY {col_name | expr | position}
      [ASC | DESC], ... [WITH ROLLUP]]
    [HAVING where_condition]
    [ORDER BY {col_name | expr | position}
      [ASC | DESC], ...]
    [LIMIT {[offset,] row_count | row_count OFFSET offset}]
    [PROCEDURE procedure_name(argument_list)]
    [INTO OUTFILE 'file_name'
        [CHARACTER SET charset_name]
        export_options
      | INTO DUMPFILE 'file_name'
      | INTO var_name [, var_name]]
    [FOR UPDATE | LOCK IN SHARE MODE]]

*Ouch*

Little bit less verbose one

<code>
SELECT column_list 
FROM Table
[JOIN friend_table ON foo = bar]
WHERE expression
GROUP BY some_groups
HAVING something
ORDER BY some_fields
LIMIT sometimes;
</code>

Columns to retrieve, expression or real colums

<code>
SELECT country, COUNT(*) as total FROM Registrant GROUP BY country ORDER BY total DESC;
SELECT * FROM Registrant LIMIT 15;
</code>

Alias some columns

<code>
SELECT country as `c`, lastname name FROM Registrant WHERE lastname = 'bar';
</code>
*in the where section you can't use alias name!!!*

using not the default database

<code>
SELECT * FROM INFORMATION_SCHEMA.TABLES;
</code>

## Where are my keys ##

priority 

<code>age < 20 AND country = 'France' OR country = 'USA'</code>

equivalent to

<code>(age < 20 AND country = 'France') or country = 'USA'</code>

*AND have higher priority over OR*

_--safe-updates_ will prevent SELECT to retrieve more than 1000 rows(if LIMIT not specified) and not execute a query with join if more than 1 million rows must be processesed 
.

## Sorting ##


	SELECT first_name FROM Registrant ORDER BY first_name DESC;
	SELECT foo,bar FROM foobar ORDER BY 1, 2; -- not advised.. 
	SELECT country, COUNT(*) FROM Registrant GROUP BY country ORDER BY COUNT(*); 


If an index is applied to the order by column, it might be faster.
ORDER BY can be used with UPDATE or DELETE

Ordering is sensible to collation for non binary strings
forcing collation


    SELECT c FROM foobar ORDER BY c COLLATE latin1_general_cs


Enum sort by virtual number

## Limiting the damages ##

<code>
	SELECT * FROM Registrant LIMIT 10; -- get first 10 rows
	SELECT * FROM Registrant LIMIT 5, 10; -- get 10 rows starting from the 6th
	SELECT * From Registrant LIMIT 10 OFFSET 5; -- syntax sugar
</code>

Skipping a lot of rows is *not a good idea*

## Being original ##

<code>
	SELECT DISTINCT first_name FROM Registrant;
	SELECT COUNT(DISTINCT first_name) FROM Registrant;
</code>

## Aggregation ##

<code>
	SELECT MIN(age), MAX(age) FROM Registrant;
	SELECT SUM(age), AVG(age) FROM Registrant;
</code>

COUNT do: 

 - *: count the number of rows
 - on column: count the number of non null
 - DISTINCT column: count number of distinct non null values
 - DISTINCT column1, column2: count non null in any columns combinaison

GROUP_CONCAT, glue together some rows using a separator, it ignores NULL
<code>
	SELECT GROUP_CONCAT(country SEPARATOR ' - ' ORDER BY country ASC) countries FROM Registrant;
	SELECT GROUP_CONCAT(DISTINCT coutry) FROM Registrant;
</code>

## Null or not Null, this is the question ##

Most of the aggregates functions ignores NULL, example AVG

count(*) is an exception since it counts total number of rows

## Grouping the sheeps ##

<code>
	SELECT country, COUNT(*) '# registrants' FROM Registratant GROUP BY country;
</code>

_Note:_ Grouping modify the sort order on MySQL

## Having a lot of things ##

Here the process to filter the data: 

1. WHERE select rows
2. GROUP BY group them
3. Aggregate function do their calculations
4. HAVING filter the unneeded groups

## Roll up man ##

<code>
	SELECT country, count(*) '# registrant per country' FROM  Registrant GROUP BY Country WITH ROLLUP;
</code>

This add a nice row with country being NULL and total is the total of registrant

## l'UNION fait la force ##

<code>
SELECT email FROM Registrant_website1
UNION
SELECT email FROM Registrant_website2
</code>

The number of columns must be the same, the name of colums from first SELECT is used
Union eliminates duplicates if you need dup *UNION ALL* 

data type could be different, MySQL would choose the bigger

<code>
(SELECT email FROM Registrant_website1)
UNION (SELECT email FROM Registrant_website2 ORDER BY email LIMIT 5)
ORDER BY email
</code>
