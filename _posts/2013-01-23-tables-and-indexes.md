---
layout: post
title: Tables and Indexes
---


## Table properties ##

Each table has a set of files: 

-    .frm: the format/scheme of the table
- MyISAM:
	- .MYD: data
	- .MYI: index data
- InnoDB: all the tables(no matter what databases they are in) are stored as ibdata1 (data dictionnary, 10MB increase), ib_logfilen (5MB increase) log file

 Starting from MySQL 5.5 InnoDB is the default storage engine over MyISAM. 

By default this engine are avaible: 

 - MyISAM
 - MERGE
 - MEMORY
 - InnoDB

 max distribution may have some other engines available... 

 the limit of table number is:

  - InnoDB: 2 billions
  - MyISAM: no limit but
 	- The filesystem has a limit
 	- The filesystem might be so slow with huge number of files in same directory
 	- you might run of disk space

If you reach the limit, you can: 

 - pick up another filesystem or OS
 - change the storage engine
 - use the MERGE engine

## Creating tables ##

There is 3 ways to create tables: 

 - specify the scheme, i.e. CREATE TABLE 
 - use the output of a SELECT query
 - copy or clone an existing table scheme

### CREATE TABLE ###

Generic definition
<code>
CREATE TABLE table_name (column_definitions);
</code>

If a table already exist an error occurs, if you don't want the error use: 
<code>
CREATE TABLE IF NOT EXISTS table_name(column_definitions);
</code>

A really cool table with the full syntax
<code>
CREATE TABLE Registrant
(
	id INT UNSIGNED NOT NULL AUTO_INCREMENT,
	last_name CHAR(30) NOT NULL,
	first_name CHAR(30) NOT NULL,
	email VARCHAR(255) NOT NULL,
	country ENUM('France', 'Japan', 'Great Britain', 'ROW'),
	PRIMARY KEY(id),
	INDEX name(last_name, first_name),
	UNIQUE email(email),
	KEY (country)
) ENGINE=InnoDB
</code>

#### Storage Engine ####

When creating a table with specified storage engine, if the storage engine is not available MySQL use the default storage engine. 

The default storage engine is specified by

- --default-storage-engine option pass to the daemon
- storage_engine variable value, can be set globally using <code>SET GLOBAL storage_engine = InnoDB</code>
- the session storage_engine variable, could be set using <code>SET SESSION storage_engine = InnoDB</code> or <code>SET storage_engine = InnoDB</code>

If MySQL couldn't use the engine specifie it would emit a warning. 

You can change a table storage engine by using: <code>ALTER TABLE table ENGINE = foobar</code>

#### Table based on existing tables ####

Method 1 From select:
<code>
CREATE TABLE CityCopy1 SELECT * FROM City;
CREATE TABLE CityCopy2 SELECT * FROM City WHERE population > 3;
CREATE TABLE CityCopy3 SELECT * FROM City WHERE 0;
</code>

Creating a table using SELECT would: 

 - reset the storage engine to default
 - not copy the indexes

Method 2 using LIKE, this will automaticly create an empty table
<code>
CREATE TABLE CityDeepCopy LIKE City
</code>

This will:

 - copy index and auto increment
 - use the same engine as the original table
 - will not:
 	- copy the extra parameter of the storage engine, i.e DATA DIRECTORY, INDEX DIRECTORY for MyISAM
 	- copy foreign key definitions

#### Playing with temporary tables ####

You can create temporaly tables in MySQL.

Those have the following properties: 

 - they are automaticly deleted when you disconnects
 - they have unique name per session, so two clients create the same temp table won't conflict
 - they take priorities over normal table, if you create a temp table using an already existing table name, you can access only the temp table
 - they can be renamed only using ALTER TABLE, *RENAME TABLE won't work*

 ### Altering tables ###

 You can alter a table in following situations:

 - adding or dropping columns
 - renaming or changing definition of a column
 - adding or droping indexes
 - renaming the table

 #### Adding or dropping columns ####

 Add 
 <code>
 ALTER TABLE Registrant ADD birth_date DATE NOT NULL AFTER email;
 ALTER TABLE Registrant ADD sid INT UNSIGNED NOT NULL FIRST;
 </code>

 drop
 <code>
 ALTER TABLE Registrant DROP sid, ADD sex ENUM('Male', 'Female') AFTER email;
 </code>

 #### Modifying columns #####

 Modify syntax, this will modify only the datatype and attributes
 <code>
 ALTER TABLE Registrant MODIFY ID BIGINT UNSIGNED NOT NULL;
 </code>

 Change syntax, allow same as modify syntax but you can rename colums
 <code>
 ALTER TABLE Registrant CHANGE email mail VARCHAR(500) NOT NULL;
</code>

### Renaming tables ###

Alter syntax
<code>
ALTER TABLE t1 RENAME t2;
</code>

Rename syntax, allows to rename multiple tables
<code>
RENAME TABLE t1 TO t2, t3 TO t3;
</code>

#### Multiple modifications ####

<code>
ALTER TABLE Registrant ADD sex ENUM('M', 'F') NOT NULL DEFAULT 'M' AFTER email, 
					   MODIFY id BIGINT UNSIGNED NOT NULL,
					   DROP country,
					   KEY (sex);
</code>

## Emptying tables ##

DELETE method 
<code>
DELETE FROM table;
</code>

Pro: 

 - permits to select what rows to delete using where
 - check constraints, foreign keys, cascade delete

Cons: 
 
 - slower than truncate

Truncate methods
<code>
TRUNCATE TABLE t;
</code>

## Indexes ##

3 main types of indexes: 

 - primary key, __must be NOT NULL__, index name *is always* PRIMARY, only **one** per table
 - unique index, __may contains multiple NULL__, if column NOT NULL same functionaly as primary jey
 - non unique

2 sub type: 

 - FULLTEX
 - SPATIAL (not in exam)

### Creating indexes ###

Method 1, in CREATE TABLE, i.e.

<code>
CREATE TABLE foo
(
	id INT UNSIGNED NOT NULL PRIMARY KEY
);

CREATE TABLE bar
(
	id INT UNSIGNED NOT NULL,
	PRIMARY KEY (id)
);

CREATE TABLE foobar
(
	foo CHAR(30) NOT NULL,
	bar CHAR(30) NOT NULL,
	INDEX (foo, bar)
);
</code>

Choosing the name of index
<code>
CREATE TABLE Country
(
	iso_code CHAR(30) NOT NULL,
	name VARCHAR(255) NOT NULL,
	UNIQUE iso_code_index (iso_code)
);
</code>

Adding to existing table
<code>
ALTER TABLE Registrant ADD PRIMARY KEY (ID);
ALTER TABLE Registrant ADD INDEX foobar (foo,bar)
</code>

other syntax:

<code>
CREATE UNIQUE INDEX Index ON Registrant (ID);
CREATE INDEX name ON Registrant (last_name(50) ASC, first_name DESC);
</code>
*Cannot be used for primary key index*

Available indexing algorithm: 

 - InnoDB, MyISAM: BTREE
 - Memory: BTREE, HASH
 - NDB: BTREE, HASH

Specifying the indexing algorithm: 
<code>
CREATE TABLE foo
(
	name CHAR(10) NOT NULL,
	value TEXT NOT NULL,
	INDEX USING HASH (name)
) Engine = MEMORY;
CREATE INDEX foobar USING HASH ON memory_table (ID);
</code>

### Dropping indexes ###

Alter style:
<code>
ALTER TABLE Registrant DROP PRIMARY KEY;
ALTER TABLE Registrant DROP INDEX name_index;
</code>

_Note_: to drop the index you have to get the index name using <code>SHOW CREATE TABLE table_name</code>

Drop style:
<code>
DROP INDEX foobar_index ON foobar;
DROP INDEX 'PRIMARY' on t; -- Note the quote because of PRIMARY being a reserved word
</code>

## Playing with meta data ##

All the meta data about databases, tables, columns, triggers, views are stored in INFORMATION_SCHEMA database

get information about a table

<code>
SELECT * FROM INFORMATION_SCHEMA.TABLES
WHERE TABLE_SCHEMA = 'database' AND TABLE_NAME = 'foobar'
</code>

you can get a list of tables present in a database by:

<code>
SHOW TABLES FROM database;
</code>

or current DB

<code>
SHOW TABLES
</code>

with patterns:

<code>
SHOW TABLES LIKE 'foo%';
</code>

get the CREATE TABLE statement

<code>
SHOW CREATE TABLE foobar;
</code>

get description of table

<code>
DESCRIBE foobar;
SHOW COLUMNS FROM foobar;
SHOW FIELDS FROM foobar;
</code>

get index info

<code>
SHOW INDEX FROM foobar
</code>

