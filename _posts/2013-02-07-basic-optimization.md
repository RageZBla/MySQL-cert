---
layout: post
title: Basic optimizations
---

# Overview #

Several strategies: 

 - indexing properly: This is true for both SELECT and UPDATE AND DELETE. You should also avoid to create unnecessary indexes.
 - the way query is written might prevent use of indexes, so rewriting queries might be an option
 - EXPLAIN give some information how the MySQL optimizer processes queries, it might a good thing to look at execution plans
 - In some case, query processing could be improved by using different approaches. This include techniques such as summary tables
 - The storage engines also be chosen wisely depending of the type of application

Reasons to be concerned about optimization: 

 - a query running faster make your application run faster. Queries also create locks so it is better to realease locks as soon as possible to not block other clients
 - a query with no or poor indexes would generate a lot more I/O. The extra overhead affects your querry but also all the other queries that the server has to process. If you optimize your query you also give more ressources for other processing. 

# Use the damn index #

When create a table, consider to set the indexes because the have ushc benefits: 

 - Indexes contain sorted values. This allow MySQL to find them more quickly. This is particulary true for joins. 
 - Indexes reduce the disk I/O. The server can use the indexes and doesn't have to load the table from disk, the indexes are also cached. In certain conditions MySQL can generate the result set of a query only using indexes

MySQL supports those indexes:
 
 - PRIMARY KEY: unique, not null
 - UNIQUE: unique, allows null
 - INDEX or KEY: not unique
 - FULLTEXT: full text search index
 - SPATIAL: not covered here

To define index you can use ALTER TABLE, CREATE INDEX. It's usually better to use ALTER TABLE and define all indexes at once, this would decrease the processing needed to construct all the indexes. 

Keep this considerations in mind when creating indexes: 

 - declare indexes NOT NULL if possible, NULL is an exceptional value and requires some decision by the server to be processed
 - avoid over indexing, make sure the index would be use in a WHERE or ORDER BY or GROUP by clause
 - Every index you creates slow down table updates since indexes need to updated so don't over use it
 - The optimizer uses a strategy to estimate how many rows will be returned by an index value, if the % is too high the optimizer might choose to do a full scan, it is not a good idea to index columns with few distinct values, indexing a column declared as ENUM('Y', 'N') would not do much good. 
 - Choose unique and non-unique wisely. 
 - index column prefix rather than the entire column, you should try to keep the index as small as possible to avoid I/O and to keep as much keys of the index in cache
 - avoid creating multiple index that overlap, you can use the leftmost index prefixes.
 - the create process itself can be optimizer if you are creating more than one index at once, prefer ALTER TABLE over CREATE INDEX
 - For MyISAM and InnoDB keep the indexes statistics up to date, the optimizer use them to make decisions

# Indexing column prefixes #

	CREATE TABLE foo
	(
		`text` VARCHAR(2048),
		INDEX(`text`)
	);

the index processing will be slow because:

 - you have to read lot of information from the disk
 - longer values take longer to compare
 - the index cache is not as effective because fewer keys can be stored

Solution use prefixes


	CREATE TABLE foo
	(
		`text` VARCHAR(2048),
		INDEX(`text`(15))
	);

Indexes get smaller and perform better but you have to be careful that the index still produce a proper amount of uniqueness. see the remark about indexes columns with few distinct values. 

You can measure how many unique values are produced by executing a query of the type: 

	SELECT COUNT(DISTINCT LEFT(`text`, n)) as 'Distinct index values',
		   COUNT(*) - COUNT(DISTINCT LEFT(`text`, n)) as 'Duplicate prefix values'
	FROM foo

You might also change a UNIQUE index to non-unique one. 

# leftmost index prefixes #

In a table having a composite index (multiple-column), you can use the leftmost index columns of the index. 

Make sure to choose wisely the order of columns so you can use leftmost prefixes as well

	CREATE TABLE foo
	(
		foo1 INT NOT NULL,
		foo2 INT NOT NULL,
		foo3 INT NOT NULL,
		INDEX(foo1, foo2, foo3)
	);

	SELECT * FROM foo WHERE foo1 = 1 AND foo2 = 2; -- Still use the index

# General query enhancement #

Please be carefull about such conditions: 

 - indexed columns can be used if you are using an function or changing the expression

 	SELECT * FROM Registrant WHERE YEAR(`created`) = 1994; -- won't use index
 	SELECT * FROM Registrant WHERE `created` BETWEEN '1994-01-01' AND '1994-12-31'; -- no processing per row use index

 - indexes particulary works on joins be sure index both join side (because inner join can be reversed)

 	SELECT * FROM foo f JOIN bar ON f.id = b.id;

 - Make sure you use the proper datatype, if an conversion happens since the processing has to happen on every row, no indexes would be used

 	WHERE id = 666; -- GOOD
 	WHERE id = '666'; -- BAD

 - In certain cases the optimizer can use the indexes even for patterns matches: 

 	WHERE name = 'ba%'; -- GOOD
 	WHERE name = '%ba%'; -- BAD

# using EXPLAIN #

Explain give you the details of the processing that MySQL use to create the result set. 

# Limiting output #

Make sure to select only the columns you need, use LIMIT if don't need all the rows, those techniques would:

 - reduce the network traffic
 - in many cases LIMIT permits the server to stop the processing or to optimize it even for sorted results

_note_: using LIMIT on a big tables won't help, you should prefer WHERE whenever you can. 

# using summary tables #

If you need data to be summarized, you can store the result of the summary in a place instead of making the processing every time you need the summary. 

It has several advantages

 - You can even use temporary tables
 - the burden of the summarization is done once or at least less times
 - the original tables don't get locked anymore
 - if the summary table is small enough you can store it in memory and avoid any disk IO

# Optimizing updates #

there is also severals techniques to optimize update as well: 

 - DELETE and UPDATE use WHERE close try to write it so it can leverage indexes
 - You can analyze a DELETE or UPDATE queries just by writing a SELECT query with same WHERE and using EXPLAIN
 - You can use multiple rows insert INSET INTO foo(foo) VALUES(1),(2),(3);
 - You can group in a transaction using InnoDB
 - For any storage, LOAD DATA INFILE is even faster then INSERT
 - You can temporaly disable index updates, LOAD DATA INFILE does in for non-unique indexes if the table is empty
 - use REPLACE when you can

# Choosing the storage engines #

MyISAM use a table level locking so it performs better for applications having a read to write ratio elevated. 

InnoDB use a row level locking, it performs good a mix of heavy update and select. 

If you are using MyISAM consider if you want efficiency of processing or disk usage. 

 - Use fixed length columns for best speed. The data file structure is easier to read but waste of space
 - use variable length better disk space usage, but the file structure is dynamic. The content could be also stored in multiple place

Using MyISAM you can also use compressed read-only tables

On InnoDB because the storage of fixed or non fixed length is implemented the same, they perform the same way, fixed might be even slower in some scenarios mostly because MySQL has to read more information from the disk.

MERGE tables can use a mix of compressed and uncompressed tables. 

# Normalization #

Normalization refers to the process of restructuring tables to elimate design problems. 

 
