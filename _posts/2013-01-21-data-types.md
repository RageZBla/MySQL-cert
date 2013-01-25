---
layout: post
title: Date Types
---

# Integer #

<table>
	<tr>
		<th>Type</th>
		<th>Storage</th>
		<th>Signed</th>
		<th>Unsigned</th>
	</tr>
	<tr>
		<td>TINYINT</td>
		<td>1 byte</td>
		<td>-128 to 127</td>
		<td>0 to 255</td>
	</tr>
	<tr>
		<td>SMALLINT</td>
		<td>2 byte</td>
		<td>–32,768 to 32,767</td>
		<td>0 to 65,535</td>
	</tr>
	<tr>
		<td>MEDIUMINT</td>
		<td>3 byte</td>
		<td>–8,388,608 to 8,388,607</td>
		<td>0 to 16,777,215</td>
	</tr>
	<tr>
		<td>INT</td>
		<td>4 bytes</td>
		<td>–2,147,683,648 to 2,147,483,647 </td>
		<td>0 to 4,294,967,295</td>
	</tr>
	<tr>
		<td>BIGINT</td>
		<td>8 bytes</td>
		<td>–9,223,372,036,854,775,808 to 9,223,372,036,854,775,807</td>
		<td>0 to 18,446,744,073,709,551,615</td>
	</tr>
</table>

The width is just optional INT(4) would not be enforced and the needed storage page does not change. 

# floating point data types # 

	weight FLOAT(10,3)
	avg_score DOUBLE(20,7)

10 digites precision, 3 decimals, simple or double precision

# fixed point data #

	cost DECIMAL(10,2)

if omitted scale and precision are 10 and 0, so this is all equivalent

	total DECIMAL,
	total DECIMAL(10)
	total DECIMAL(10,0)

amount of storage depends on scale and precision about 4 bytes per 9 digits

NUMERIC is a synonym for DECIMAL. 

# BIT #

	bit_col1 BIT(4)

up to 64

0 to 2**n - 1
	
	SELECT b'1111'

# String #

	SHOW COLLATION LIKE 'latin1%';

# non-binary string #

They have a charset and a collation

	CREATE TABLE Registrant
	(
		last_name CHAR(32) NOT NULL CHARACTER SET utf8 COLLATE utf8_general_cs,
		first_name CHAR(32) NOT NULL CHARACTER SET utf8,
	)

<table>
	<tr>
		<th>Type</th>
		<th>Storage</th>
		<th>Maximum length</th>
	</tr>
	<tr>
		<td>CHAR(M)</td>
		<td>M characters</td>
		<td>255 characters</td>
	</tr>
	<tr>
		<td>VARCHAR(M)</td>
		<td>M characters + 1 or 2 bytes</td>
		<td>65,535 characters</td>
	</tr>
	<tr>
		<td>TINYTEXT</td>
		<td>L characters + 1 byte</td>
		<td>255 characters</td>
	</tr>
	<tr>
		<td>TEXT</td>
		<td>L characters + 2 bytes</td>
		<td>65,535 characters</td>
	</tr>
	<tr>
		<td>MEDUIMTEXT</td>
		<td>L characters + 3 bytes</td>
		<td>16,777,215 characters</td>
	</tr>
	<tr>
		<td>LONGTEXT</td>
		<td>L characters + 4 bytes</td>
		<td>4,294,967,295 characters</td>
	</tr>
</table>

# binary string #

No collation, so funny result with string functions

<table>
	<tr>
		<th>Type</th>
		<th>Storage</th>
		<th>Maximum length</th>
	</tr>
	<tr>
		<td>BINARY(M)</td>
		<td>M bytes</td>
		<td>255 bytes</td>
	</tr>
	<tr>
		<td>VARBINARY(M)</td>
		<td>M bytes + 1 or 2 bytes</td>
		<td>65,535 bytes</td>
	</tr>
	<tr>
		<td>TINYBLOB</td>
		<td>L bytes + 1 byte</td>
		<td>255 bytes</td>
	</tr>
	<tr>
		<td>BLOB</td>
		<td>L bytes + 2 bytes</td>
		<td>65,535 bytes</td>
	</tr>
	<tr>
		<td>MEDUIMBLOB</td>
		<td>L bytes + 3 bytes</td>
		<td>16,777,215 byes</td>
	</tr>
	<tr>
		<td>LONGBLOB</td>
		<td>L bytes + 4 bytes</td>
		<td>4,294,967,295 bytes</td>
	</tr>
</table>

# ENUM and set #

an ENUM may contain up to 65,535 and require 1 or 2 byte of storage

Value of enum are stored as integer starting from 1, 0 is used to point invalid value

	CREATE TABLE foo
	(
		foo ENUM('foo','bar')
	)

SET use bit encoding to represent values, up to 64 members, need 1 to 8 bytes)

	CREATE TABLE foo
	(
		foo SET('foo','bar')
	)

# Temporal #

<table>
	<tr>
		<th>Type</th>
		<th>Storage</th>
		<th>Range</th>
	</tr>
	<tr>
		<td>DATE</td>
		<td>3 bytes</td>
		<td>'1000-01-01' to '9999-12-31'</td>
	</tr>
	<tr>
		<td>TIME</td>
		<td>3 bytes</td>
		<td>'838:59:59' to '838:59:59'</td>
	</tr>
	<tr>
		<td>DATETIME</td>
		<td>8 bytes</td>
		<td>'1000-01-01 00:00:00' to '9999-12-31 23:59:59'</td>
	</tr>
	<tr>
		<td>TIMESTAMP</td>
		<td>4 bytes</td>
		<td>'1970-01-01 00:00:00' to mid 2037</td>
	</tr>
	<tr>
		<td>YEAR</td>
		<td>1 byte</td>
		<td>1901 to 2155 -> YEAR(4)<br/>1970 to 2069 -> YEAR(2)</td>
	</tr>
</table>

zero value exist for storing invalid value: 
	- 0000-00-00
	- 00:00:00

Format is ISO8601: YYYY-MM-DD

delimiter can be replace by / 

2 digits year are converted to 4 but be careful of 70 to 99 and 00 to 69, i.e 1970 versus 2070

time format is 'hh:mm:ss'

# Timestamp #

Unix tiomestamp from 1970-01-01 00:00:00 UTC

DEFAULT CURRENT_TIMESTAMP and ON UPDATE CURRENT_TIMESTAMP

For compatibity MySQL asign both flags to first timestemp column of table

You can use DEFAULT and ON update to different columns to emulate that use DEFAULT 0 or DEFAULT NULL

	CREATE TABLE foobar
	(
		id INT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
		created TIMESTAMP DEFAULT 0,
		updated TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
	);

By default timestamp column are NOT NULL

# Per connection time zone #

3 formats available: 

 - '+hh:mm'
 - 'US/Eastern' need the mysql database to be loaded
 - SYSTEM is OS timezone (default)

 	SELECT @@global.time_zone, @@session.time_zone;

 	SET time_zone = '+00:00'

 Timestamp value are internally stored in UTC, mysql convert depending on session time zone. 

 	SELECT CONVERT_TZ('2012-01-01 08:00:00', '+01:00', '+09:00');

 # numeric attributes # 

 attributes:
 	- UNSIGNED: double range, no negative
 	- ZEROFILL: 00010
 	- AUTO_INCREMENT: auto increment!

 # String attributes #

 	- CHARACTER SET
 	- COLLATE
 	- BINARY: binary collation

 # General attributes #

 	- NULL, NOT NULL
 	- DEFAULT: must be constant, cannot be used on TEXT and BLOB and AUTO_INCREMENT

 default value when no default: 

 	- integer: 0
 	- string: ''
 	- enum: first member
 	- time/data: '0000-00-00' or '00:00:00'

# Auto increment #

	LAST_INSERT_ID()

is connection specifics

must be 
 
  - integer data type
  - positives values so make sense to use in conjuction UNSIGNED
  - should be indexed UNIQUE or PRIMARY KEY or INDEX
  - NOT NULL

a positive value will raise the count

auto_increment doesn't work with UPDATE

NO_AUTO_VALUE_ON_ZERO would force NULL to auto increment, 0 also works without this mode

*Capping*

MyISAM allow composite indexes that includ AUTO_INCREMENT

# Handling missing values #

MySQL missing values with such process: 

 - if DEFAULT is present use default
 - if no default: 
    - no strict mode, insert implicit default value
    - strict mode, transaction table -> error, if transactional and multi insert/update -> warning (2 or later rows) (*avoid partial updates*)

	SHOW WARNINGS

Conversion of out of range, for example TINYINT = 128 -> 127

String truncation

data type default, '0000-00-00'

assigning NULL to NOT NULL, 1 statement -> error, multiple insert default type value + warning

In strict mode, transaction get cancelled, on non transaction table, if first row error, if second or more default value and warning

STRICT_ALL_TABLES cancel no matter what, kind of dangerous for non transactional


# more checking #

	SET sql_mode = ‘STRICT_ALL_TABLES,ERROR_FOR_DIVISION_BY_ZERO’;

	SET sql_mode = ‘STRICT_ALL_TABLES,NO_ZERO_DATE,NO_ZERO_IN_DATE’;

# overriding #

	SET sql_mode = ‘ALLOW_INVALID_DATES’;
