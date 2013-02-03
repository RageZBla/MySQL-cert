---
layout: post
title: Importing and exporting data
---

# Import and export operations #

MySQL includes two statements to import or export data from or in file: 

 - LOAD DATA INFILE: read data from a file and insert them in a table
 - SELECT ... INTO OUTFILE: write results of a select to a file

# Importing and exporting using SQL #

	LOAD DATA INFILE 'file_name' INTO TABLE table_name;

By default MySQL assumes: 
 
 - the file is on the server host
 - fields are separated by tab characters
 - line are terminated by single \n 

 But the syntax give you possibilities to play with such parameters: 

  - table to load
  - name and location of file
  - whether to ignore linen at the beginning of the file
  - which columns to load
  - whether to skip or transform data values before loading them
  - how to handle duplicates
  - the format of the file 

Full syntax is 

	LOAD DATA [LOCAL] INFILE 'filename'
	[IGNORE | REPLACE]
	INTO TABLE table_name
	format_specifiers
	[IGNORE n lines]
	[(column_list)]
	[SET (assugnment_list)]

# File location #

By default MySQL assume that the file is on the server side. 

If LOCAL is used, MySQL read the file from the client side. 

Without local such rules applies: 

 - if you use full path, MySQL use it as is
 - if you specify a relative name with single component, MySQL looks into the default database directory
 - if you specify a realitve path with more than one component, MySQL search relative from the data directory

supposing the data directory is /var/mysql/data, and default database is test, such statements are equivalent

 	LOAD DATA INFILE '/var/mysql/data/test/data.txt' INTO table t;
 	LOAD DATA INFILE 'data.txt' INTO table t;
 	LOAD DATA INFILE './test/data.txt' INTO table t;

In case of LOCAL being used: 

	- full path is applied as is
	- relative path looks relative to current directory

# Skipping lines #

	LOAD DATA INFILE '/tmp/data.txt' INTO TABLE t IGNORE 1 lines;

# Loading specific table columns # 

	LOAD DATA INFILE '/tmp/registrant.txt' INTO TABLE Registrant(firstname, lastname, email, email_format);

You can specify the order as well.

# Skipping or transforming column values #

	LOAD DATA INFILE '/tmp/registrant.txt' INTO TABLE Registrant(firstname, lastname, @email, email_format) SET email = LOWER(@email)

# Duplicate records #

When importing from server side: 

 - By default, duplicate records generate an error, the record inserted before are left in the table and the rest of the file is not read
 - if you specify IGNORE after the filename, MySQL ignore duplicate records and process the whole file 
 - if you specify REPLACE after the filename, MySQL use the same behavior as REPLACE statement

IGNORE and REPLACE keywords are mutually exclusives.

For importation from client side, MySQL ignores duplicate records and use same processing as IGNORE, this is because MySQL can't stop a transfer between the client and server. 

# Extra info #

LOAD DATA INFILE return a string such as

	Records: 174 Deleted: 0 Skipped: 1 Warning: 14

Fields explanations: 

	- Records: indicate the number of record parsed from file
	- Deleted: indicate the number of records deleted from table, same as REPLACE
	- Skipped: indicate the number of lines in the files that have been ignored because of duplicate records
	- Warnings: number of warnings, conversion, missing data etc... it is per column so can be greater than the number of records

# Privileges for LOAD DATA INFILE #

LOAD DATA INFILE requires INSERT privilege, DELETE privileges if REPLACE is used. For an importation from server side you must have file FILE privilege. FILE is an administrative privilege so you might need a root account... 

# LOAD DATA INFILE efficiency #

Globally LOAD DATA INFILE is more efficient than INSERT. 

# Exporting data #

Basic syntax:

	SELECT * INTO OUTFILE 'Registrant.txt' FROM Country

On the contrary of LOAD DATA INFILE, OUTFILE allow you only to write files on the server host. 

It uses the sames rule to determine path as LOAD DATA INFILE. 

There is few restrictions: 

 - the destination files have to not exist, this is to prevent some hacker stuff
 - you must have FILE privilege
 - File create is owned by MySQL user account but is world writeable
 - By default the output contain one line per record, delimited by tab and with \n as line delimiter, this can be changes using the format specifiers

 The fact it only writes on server side have few implications: 

  - You need a ssh account if you want to retrieve the file, or if you can put the file just to use in a subsequen LOAD DATA INFILE
  - the file being world readable, it is not a good idea to use that with sensitive information
  - you need the administrator to remove the file from the server because file are owned by MySQL

# Format specifiers #

	FIELDS 
		TERMINED BY 'string',
		ENCLOSED BY 'char'
		ESCAPED BY 'char'
	LINES TERMINATED BY 'string'

Default values are: 

 - separator TAB
 - no quote, for LOAD DATA INFILE quote are stripped, OPTIONALLY ENCLOSED BY also works
 - line terminator is \n
 - default escape is \, escaped stuff: 
   - \N: NULL
   - \0: NULL 0 byte
   - \b: baskspace
   - \n: new line
   - \r: carriage return
   - \s: space
   - \t: tab
   - \': single quote
   - \": double quote
   - \\: back slash  

\N is understood if it only appears alone.... 

examples: 

	LOAD DATA INFILE '/tmp/registrant.txt' INTO TABLE Reg
	FIELDS TERMINATED BY ',' ENCLOSED BY '"' LINES TERMINATED BY '\n\r';

more: 

	SELECT * INTO OUTFILE '/tmp/super-reg.txt' 
	FIELDS TERMINATED BY ',' ENCLOSED BY '"' LINES TERMINATED '\n' 
	FROM t

# Importing and exporting NULL #

 - for LOAD DATA INFILE a single \N unquoted is interpreted as NULL
 - SELECT INTO OUTFILE use \n as well

# Importing and exporting form command line #

## mysqlimport ##

	mysqlimport otpions db_name input_file

mysqlimport use the filename to determine in what table you want to import. To do so, mysqlimport strip out the extension and use that as table name. 

Since mysqlimport is closely linked to LOAD DATA INFILE it take the same options: 

 - --lines-termined-by=string
 - --fields-terminated-by=string
 - --fields-enclosed-by=char
 - --field-escaped-by=char
 - --ignore 
 - --replace
 - --local

## Exporting data using mysqldump ##

mysqldump can write SQL or tab delimited files, here we are only interested in the latter. 

	mysqldump --tab=dir_name options db_name tbl_name [tbl_name2]
	mysqldump -T dirname options db_name tbl_name

mysqldump create two files 

 - table.sql, on client side, owned by you, SQL to create the table
 - table.txt, on server side, contain data

 You need to have FILE privileges. If you don't want the sql file use --no-create-info

 mysqldump take such format options: 

  - --lines-terminated-by=string
  - --fields-terminated-by=string
  - --fields-enclosed-by=char --fields-optionally-enclosed-by=char
  - --fields-escaped-by=char

The --tab cannot be used with --all-databases and --databases options. 

If you don't specify a table, mysqldump dump all the tables in specified database