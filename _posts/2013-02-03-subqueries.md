---
layout: post
title: Subqueries
---

# Type of subqueries #

4 categories: 

 - Scalar subqueries: return a single row and single column
 - row subqueries: return a single row with multiple columns
 - column subqueries: return multiples rows with a single column
 - table subqueries: return multiples rows with multiple columns

In some cases sub queries can be rewritten using joins. 

The support for subqueries was added in MySQL 4.1. 

# Scalar expression #

	CREATE TABLE t1 (s1 INT, s2 CHAR(5) NOT NULL);
	INSERT INTO t1 VALUES(100, 'abcde');
	SELECT (SELECT s2 FROM t1);

	CREATE TABLE t1 (s1 INT);
	CREATE TABLE t2 (s1 INT);
	INSERT INTO t1 VALUES(5);
	INSERT INTO t2 VALUES(500);
	SELECT (SELECT SUM(s1) FROM t1) / (SELECT SUM(s1) FROM t2) as ratio;

# Correlated subqueries #

 - A non correlated subquery is a query where the subquery is not dependent to the outer query
 - on the contrary correlated subquery the subquery reference some columns from the outer query


# Comparing subquery results to out query columns #

Normally you can use comparaison operators(=, <>, >, >=, <, <=) if the subquery don't return a scalar

	CREATE TABLE t1 (s1 INT);
	CREATE TABLE t2 (s1 INT);
	INSERT INTO t1 VALUES(5);
	INSERT INTO t2 VALUES(500),(250);
	SELECT * FROM t1 WHERE s1 >= (SELECT s1 FROM t2);

would produce this error <code>ERROR 1242: Subquery returns more than 1 row</code>

# Using ALL, ANY and SOME #

	SELECT * FROM t1 WHERE s1 >= ALL (SELECT s1 FROM t2);

All: check that all the result matches the test
Any, Some: are identical, check if 1 or more results match the test

# Exists #

Check if the subquery return any rows... 

# Row subqueries #

	SELECT foo FROM foo WHERE (foo, bar) = (SELECT foo, bar FROM bar WHERE bar = 'foo');
	SELECT foo FROM foo WHERE ROW(foo, bar) = (SELECT foo, bar FROM bar WHERE bar = 'foo');

# subqueries in FROM statement #

	SELECT foo FROM (SELECT foo from bar) as t;

This form cannot be correlated! 

# Using sub queries in update #

You cannot update or delete from a table used both in the outer and subquery statement

	DELETE FROM NACities WHERE ID IN (SELECT ID FROM NACities WHERE Population < 500)

It would create an error

