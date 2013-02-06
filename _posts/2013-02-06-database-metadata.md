---
layout: post
title: Database metadata
---

# overview #

MySQL produce some metadata.

Most of the SHOW statements rely most of those metadata structure. 

mysqlshow act as a wrapper around SHOW

INFORMATION_SCHEMA database is implemented, INFORMATIOB_SCHEMA is a standard SQL. 

SHOW also provide a WHERE and LIKE. 

# using metadata #

	SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES
	WHERE TABLE_SCHEMA = 'INFORMATION_SCHEMA'
	ORDER BY TABLE_NAME;

Here a list of all table in the virtual database INFORMATION_SCHEMA

 - CHARACTER_SETS: information about character sets
 - COLLATIONS: information about collations for each character set
 - COLLATION_CHARSET_SET_APPLICABILITY: information about which character set applies to each collation
 - COLUMNS: information about columns
 - COLUMN_PRIVILEGES: information about column level privileges
 - KEY_COLUM_USAGE: information about constraints on columns
 - ROUTINES: functions and procedures informations
 - SCHEMATA: databases
 - SCHEMA_PRIVILEGES: database level privileges
 - STATISTICS: table indexes statistics
 - TABLES: tables information
 - TABLE_CONSTRAINTS: constraints on tables
 - TABLE_PRIVILEGES: table leve privileges
 - TRIGGERS: triggres
 - USER_PRIVILEGES: server wide privileges
 - VIEWS

You can't update the INFORMATION_SCHEMA 'database'. Appart from that you can use all the SELECT your want! 

# using SHOW and DESCRIBE #

	SHOW DATABASES;
	SHOW DATABASES LIKE 'w%';
	SHOW TABLES;
	SHOW FULL TABLES;
	SHOW TABLES FROM db;
	SHOW COLUMNS FROM table;
	SHOW FIELDS FROM table;
	SHOW KEYS FROM table;
	SHOW COLUMNS FROM table WHERE `Default` IS NULL;
	SHOW CHARACTER SET;
	SHOW COLLATION;
	DESCRIBE table;

# using mysqlshow #

Show database
	
	mysqlshow

show tables

	mysqlshow db

show columns

	mysqlshow db table

show only one columns

	mysqlshow db table column

show like on unix

	mysqlshow 'w%'

on windows

	nysqlshow 'w*'

? and _ also matches a single character