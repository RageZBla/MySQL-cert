---
layout: post
title: Databases
---

# Intro #

 - The server reprensents database as directories
 - default character set and collation are associated with a database
 - databases cannot be nested

# Creating #

Lambda syntax

	CREATE DATABASE mydb;

If not exists

	CREATE DATABASE IF NOT EXISTS mydb;

Character set and collate

	CREATE DATABASE mydb CHARACTER SET utf8 COLLATE utf8_danish_ci;

Use

	USE mydb;

# Altering #

Change character set, collation

	ALTER DATABASE mydb COLLATE utf8_polish_ci;

*Cannot rename need to do a dump restors*

ALTER DATABASE name of database is optional, if not specified works on default db

# Dropping #

normal: 

	DROP DATABASE mydb;

if not exists

	DROP DATABASE IF EXISTS mydb;


# Meta #

	SELECT * FROM INFORMATION_SCHEMA.SCHEMATA WHERE SCHEMA_NAME = ‘foo’;
	SHOW DATABASES;
	SHOW DATABASES LIKE 'f%';