---
layout: post
title: Prepared statements
---

# Benefits #

Prepared statements give a performance boost because MySQL have to parse the SQL query only once. 

They are also convenient when you need to execute multiple time some queries with only different data.

# Syntax #

	PREPARE my_stmt FROM 
	'SELECT COUNT(*) FROM foobar where bar = ? AND foo = ?';
	SET @foo = 'bla', @bar = 1;
	EXECUTE my_stmt USING @foo, @bar;
	DEALLOCATE PREPARE my_stmt;

# Preparing a statement #

	SET @sql = 'SELECT COUNT(*) FROM foobar WHERE bar = ?';
	PREPARE  stmt FROM @sql;

If a prepared statment already exists and you try to redefine it, MySQL first delete the previous statement and create a new one. 

In case of error the statement is not created. So if you use an already existing name and the statement failed, the old statement is lost.

Prepared statement can be used in such conditions: 

 - SELECT statements
 - INSERT, REPLACE, UPDATE and DELETE statements
 - CREATE TABLE statements
 - SET, DO, and many SHOW statements

A statement is linked with a connection, when the connection is close the statements are discarded. 

# Executing a prepared statements #

You have to provide value to replace the '?' in the query. 

	SET @var = 'foo';
	EXECUTE stmt USING @var;

# Deallocating prepared statement #

	DEALLOCATE PREPARE stmt;
	DROP PREPARE stmt;

DROP PREPARE also work. 

