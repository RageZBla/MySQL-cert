---
layout: post
title: SQL Expressions
---


## Numeric expressions ##

Exact value number versus approximate value

<code>
	SELECT 1.1 + 2.2 = 3.3, 1.1E0 + 2.2E0 = 3.3E0; -- 1, 0
</code>

## String expression ##

Single or double quote can be used to quote strings expressions. If ANSI_QUOTES is enabled double quotes can be used only for identifiers

Result of comparaisons like =, <>, <, BETWEEN are dependent of Binary or non binary and for non binary the collation(case sensivility, transliterations).

<code>
 SELECT CONCAT('foo', 'bar', REPEAT('@', 3));
</code>

The || can be used if PIPES_AS_CONCAT is enable, in normal mode it is a logical or
<code>
	SELECT 'foo' || 'bar'
</code>

### Case sensivity ###

The default character set for constant string expression is determined by variables character_set_connection and collation_connection. 

<code>
	SELECT 'Hello' = 'hello'
</code>

If on side of comparaison is a binary string both are treated as binary
<code>
	SELECT BINARY 'foo' = 'FOO'; -- niet
</code>

LENGTH returns the length of a string in bytes; 
CHAR_LENGTH return the length in characters
UPPER and LOWER only works on non-binary

to convert a binary string to a non binary, use convert
<code>
	SELECT UPPER(CONVERT(BINARY 'foo' USING utf8))
</code>

MD5 calculate a 32 byte checksum, so everything is binary

### Like it or not ### 

% matches 0 to n characters
_ matches a single character

LIKE evaluate to null if any operands is NULL

<code>
SELECT 'ABC' like 'abc', 'ABC' LIKE BINARY 'abc'; -- true, false
</code>

LIKE is allowed with non string

<code>
	SELECT 1900 LIKE '19%';
</code>

using a table column is valid

<code>
	SELECT * from bar WHERE a LIKE b;
</code>

escape is by default \

<code> 
	SELECT 'AA' LIKE 'A\%'
</code>

the escape character could be change

<code>
	SELECT 'AA' LIKE 'A@%' ESCAPE '@'
</code>

## Time matters ##

interval arithmetic

<code>
SELECT '2010-01-01' + INTERVAL 10 DAY, INTERVAL 10 DAY + '2012-01-01';
</code>

## Null ##

NULL operations always return NULL, NULL propage

<code>
	SELECT NULL + 1, NULL + 2
<code>

You can't use = and <> operators, you have to use IS NULL or IS NOT NULL
<code>
	SELECT NULL IS NULL, NULL IS NO NULL
</code>

There is a special operand in MySQL

<code>
	SELECT 1 <=> NULL, 0 <=> NULL, NULL <=> NULL;
</code>

ORDER BY, GROUP BY, DISTINCT consider NULL values are identical. 

If ERROR_FOR_DIVISION_BY_ZERO is not enabled, 1/0 returns NULL

## Functions ##

PI returns PI

LEAST return the smaller number of a list
<code>
	SELECT LEAST(1, 2, -5, 10) = -5, LEAST('cdef', 'ab', 'ghi') = 'ab';
</code>

GREATEST do the reverse of least take the bigger value

INTERVAL(trigger, values1, values2) return the number of items that <= , i.e. value_n <= trigger

IN ('a', 'b', 'c') is equivalent to a = 'a' OR a = 'b' or a = 'c'

BETWEEN 5 AND 10 is equivalent to a >= 5 AND a <= 10

### Control the flow ###

IF(expresion, true_expr, false_expr), i.e.

<code>
	SELECT IF(1 > 0, 'yes', 'no')
</code>

<code>
	CASE case_expr
		WHEN when_epr THEN result
		[WHEN when_expr THEN result]
		[ELSE result]
	END
</code>

second syntax

<code>
	CASE 
		WHEN when_expr THEN result
		[WHEN when_expr THEN result]
		[ELSE result]
	END
</code>

## Maths sucks ##

ROUND if fraction >= .5 go to next number if less decrease

Round on float is funny
<code>
	SELECT ROUND(2.85E1) = 28, ROUND(2.85E1) = -28
<code>

FLOOR and CEILING

<code>
	SELECT FLOOR(-14.7) = -15, FLOOR(14.7) = 14;
	SELECT CEILING(-14.7) = -14, CEILING(14.7) = 15;
</code>

ABS AND SIGN

<code>
	SELECT ABS(-66) = 66;
	SELECT SIGN(-1) = -1, SIGN(0) = 0, SIGN(1) = 1;
</code>

SIN, TAN, COS

<code>
	SELECT DEGREE(PI()) = 180, PI() = RADIAN(180)
</code>

RAND is random number between 0 and 1

### String functionns ###

LENGTH = number of by
CHAR_LENGTH = number of charactrs

CONCAT concatenate, concat propagate NULL

<code>
	SELECT CONCAT('foo', NULL, 'bar') IS NULL
</code>

CONCAT_WS use separator

<code>
	SELECT CONCAT_WS(',', 'foo', 'bar') = 'foo,bar';
</code>

STRCMP is the same function as in C, PHP, etc... 

PASSWORD is an hash function and should not be used... 

ENCODE and DECODE
DES_ENCRYPT and DES_DECRYPT
AES_ENCRYPT and AES_DECRYPT

### Temporal functions ###

YEAR, MONTH, DAYOFMONTH
DAYOFYEAR

HOUR, MINUTE, SECOND

MAKEDATE compose a date from year and number of day in year

<code>
	SELECT MAKEDATE(2012, 106) = '2012-04-16';
</code>

MAKETIME

<code>
	SELECT MAKETIME(21, 30, 10) = '21:30:10';
</code>

CURRENT_DATE, CURRENT_TIME, CURRENT_TIMESTAMP

## Null Funcs ##

ISNULL

IFNULL()

## Comments in SQL ##

Shell style comment # - single line 

C style /* */ multi line

SQL style -- space is needed single line

/*! style comment get executed

/*!50002 bar  get executed if the version is >= 

