---
layout: post
title: Updating data
---

# INSERT #

Syntax 1

<code>
INSERT INTO table (columns) VALUES(values);
</code>

Syntax 2

<code>
	INSERT INTO table SET foo = 'bar', bar = 'foo';
</code>

This is valid but rely solely on default value

<code>
	INSERT INTO foobar() VALUES();
	
</code>

not specifying columns, should insert in same order as DESCRIBE, should specify all the columns!

<code>
	INSERT INTO foobar VALUES('foo', 'bar');
</code>

multiple insert

<code>
	INSERT INTO foobar VALUES('foo', 'bar'),('bar', 'foo');
</code>

INSERT IGNORE helps to handle duplicte keys, MySQL skip those records

ON DUPLICE KEY UPDATE magic

<code>
	INSERT INTO log(userid, location, counter) VALUES(1, 'office', 1) ON DUPLICATE KEY UPDATE counter = counter + 1
</code>

2 rows affected means that a record get unpdated because of ON DUPLICATE KEY UPDATE

# REPLACE #

equivalent of insert or update. To be exact if no key violation happens REPLACE just insert, if any key violation happens REPLACE delete the duplicates ROWS and insert the new one. 

*Replace can delete more than one row, if there is multiple index violation*

REPLACE use the same syntax has insert... 

# UPDATE #

UPDATE with same values, affects no rows and don't update the auto update timestamps

UPDATE can use ORDER BY and LIMIT

<code>
	UPDATE foobar SET id = id - 1 ORDER BY id;
</code>

--safe-updates won't allow UPDATE with no where, if LIMIT clause is present won't apply 

# Delete and truncate #

DELETE return the true number of record deleted

TRUNCATE and DELETE with no where reset the auto increment counter to 1

DELETE with no where is equivalent of TRUNCATE, DELETE WHERE 1 executes nuch more slowly

# Privileges #

INSERT and DELETE and UPDATE require privileges with same name
REPLACE requires INSERT and DELETE
TRUNCATE requires DELETE
