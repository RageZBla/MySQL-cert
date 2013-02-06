---
layout: post
title: Stored procedures and functions
---

# Canvea #

Stored procedures and triggers can cause issues with replication and binary logs. This is particulary dangerous, if your routines have non deterministic behavior. 

If you want to use both binary logging and stored procedure, you should start the server using --log-bin-trust-routine-creators or at runtime using: 

	SET GLOBAL log_bin_trust_routine_creators = 1;

# Benefits #

 - More flexible SQL syntax: you can use control flow, loops, express complex logic
 - Error handling capabilities: you can control what happen in case of error
 - Standard compliance: MySQL is using the standard SQL syntax, so the routines could be ported to another RDBM
 - Code packaging: the code is as near from the data and can be used by multiple applications, you have to recode less logic
 - less "re-invention of the wheel": a collection of rountines act as a library, developers can use those routine "off the shelf", this facilitate sharing knowledge
 - Separation of logic: factoring out some business logic in the database routines could simplify the logic of the application
 - ease of maintenance: since the routine logic is attached to the database, it easier to maintain and upgrade
 - reduction of bandwidth requirements: the routine can run as many queries, without sending results to the client, it spare the bandwidth(to be noted that it would load the database server)
 - server upgrades benefit the clients, because the processing is happening on the server
 - better security: using routine you can show only needed data, also because of the rountine delegation you can granulate your security model

# differences between procedures and functions #

 - a procedure does not return value and accepts paremeters as both IN and OUT 
 - functions are invoked in the same fashion as builtin MySQL functions and returns a scalar
 - you have to use CALL to invoke procedure
 - procedures can contain transactions such as COMMIT, ROLLBACK
 - can contain DDL statement such as CREATE, DROP
 - function can be called with spaces i.e. foo ()
 - USE can't be used in routines
 - routines are executed with the sql_mode as when they were created.



Procedures can generate a or multiple result sets. 

# namespace for stored routines #

 - routines are attached to a database, if you don't fully qualify the routine MySQL use the default database. 
 - when a routine executes, MySQL change the default database to the routine database after the routine finishes MySQL revert the default database. 
 - when you drop a database all the associated routines are dropped

store procedures and functions do not share the same namespace, you can define a function and a procedure with same name. 

# defining stored routines #

	delimiter //
	CREATE PROCEDURE foobar_count ()
	BEGIN
		SELECT 'foo', count(*) FROM foo;
	END;
	delimiter ;

# creating stored routines #

	CREATE 
    [DEFINER = { user | CURRENT_USER }]
	PROCEDURE proc_name ([parameters])
		[characteristics]
		routine_body

	CREATE 
	[DEFINER = { user | CURRENT_USER }]
	FUNCTION func_name ([parameters])
		RETURNS data_type
		[characteristics]
		routine_body

If the routine doesn't have a fully qualified name it is created in the default database. When invoked the procedure, unqualified table are using the routine database, if you need tables from other database, use a fully qualified name.

The characteristics clause contains: 

 - SQL SECURITY {DEFINER | INVOKER} 
    - Definer: use the privileges of the user who created the routine, default
    - Invoker: use the privileges of the user invoking the routine
    - this parameters enable you to delegate privileges
 - DETERMINISTIC or NOT DETERMINISTIC
    - MySQL don't verify, so be honest
    - choosing the wrong type could lead to some performance penalty, the optimizer to make incorect execution plan.
 - LANGUAGE SQL
 	- specify the language use in the routine
 	- now only SQL is supported
 - COMMENT
  	- a comment explaining what the routine is doing

examples: 

	CREATE PROCEDURE surface (width INT, height INT)
		SELECT width * height

	CREATE FUNCTION surface (width INT)
		RETURNS FLOAT
		RETURN width * height;

# Compound statements #

Stored routines requires that a routine is a single statement, this is not quite true because you can use compound statement. 

example: 

	CREATE PROCEDURE foobar_count ()
	BEGIN
		SELECT 'foo', count(*) FROM foo;
	END;

The end must be followed by a ';', BEGIN does not! 

Block can be labeled: 

	label: BEGIN
		SELECT 'foo';
	END label;

label can also be use with LOOP, REPEAT, WHILE constructions. 

# declaring parameters #

for procedures parameters can be declared as: 

 - IN: the parameter is passed to the routine, the routine can modify the value but the modification can't be seen by the caller
 - OUT: assigned to NULL when the routines start, routines modifies it, the caller can see the new value
 - INOUT: combo! 

by default parameters use IN as modifier.

example:

	CREATE PROCEDURE test_param (IN p_in INT, OUT p_out INT, INOUT p_inout INT)
	BEGIN
		SET p_in = 1, p_out = 2, p_inout = 3;
	END;

# DECLARE statement #

You should use declare to "declare" local variable, conditions, handlers for conditions, cursors. You should declare them in this order: 

 1. local variables
 2. conditions
 3. handlers
 4. cursors

# variables #

	DECLARE var_name [, var_name] data_type [DEFAULT value]

# assigning variables #

	DECLARE var1, var2, var3
	SET var1 = 1;
	SELECT foo INTO var2 FROM foo WHERE bar = 'foo';

The select must returns a single row. 

# conditions and handlers #

handles duplicat record error:

	DECLARE EXIT HANDLER FOR SQLSTATE '23000'

	DECLARE dup_key CONDITION FOR SQLSTATE '23000';
	DECLARE null_not_allowed CONDITION FOR 1048; -- mysql error code

	DECLARE handler_type HANDLER FOR condition_type [, condition_type]
	statement

Each condition associated with an handler must be one of the following: 

 - an SQLSTATE value or MySQL error code
 - a condition name declared previously
 - SQLWARNING, handle all the conditions for SQLSTATE values beginning with 01
 - NOT FOUND, all the SQLSTATE beginning with 02
 - SQLEXCEPTION: handle both SQLWARNING and NOT FOUND

You can set an handler to skip the statement matching the condition

	DECLARE CONTINUE HANDLER FOR SQLSTATE '02000' SET foo = 1;

# Cursors #

Cursor in MySQL are:
 
 - are not updatable
 - are not scrollable, you cannot rewing or skip some rows


 BEGIN
 	DECLARE row_count INT DEFAULT 0;
 	DECLARE foo CHAR(3);
 	DECLARE c CURSOR FOR select foo FROM bar;
 	OPEN c;
 	BEGIN
 		DECLARE EXIT HANDLER FOR SQLSTATE '02000' BEGIN END;
 		LOOP
 			FETCH c INTO foo;
 			SET row_count = row_count + 1;
 		END LOOP;
 	END;
 	CLOSE c;
 	SELECT 'number of rows = ', row_count;
 END;


# retrieving multiple result sets #

Stored procedures call execute multiple SELECT queries on behalf of the client

	CREATE PROCEDURE multi_select ()
	BEGIN
		select 'foo';
		select 'bar';
		select 'foobar';
	END;
	CALL multi_select();

# Flow control #

	IF expr
		THEN statement_list
		ELSEIF expr THEN statement_list
		[ELSE statement_list
	END IF; 

	CASE case_expr
		WHEN when_expr THEN statement_list
		[WHEN when_expr THEN statement_list]
		[ELSE statement_list]
	END CASE;

	CASE
		WHEN expr THEN statement_list
		[WHEN expr THEN statement_list]
		[ELSE statement_list]
	END CASE;

# loop construction #

loop construct never finishes, you have to use an handler or the LEAVE instruction or RETURN

	LOOP
		statement_list
	END LOOP;

	DECLARE i INT DEFAULT 0;
	my_loop: LOOP
		SET i = i + 1;
		IF i >= 10 THEN
			LEAVE my_loop;
		END IF;
	END LOOP my_loop;

	REPEAT
		statement_list
	UNTIL expr
	END REPEAT;

	WHILE expr DO
		statement_list
	END WHILE;

# tranfer of control #

LEAVE can finish a block or a loop
ITERATE goes back to the beginning of the block

# altering routines #

You can alter the routine characteristics using: 

	ALTER PROCEDURE proc_nmae [characteristics];
	ALTER FUNCTION func_name [characteristics];

You can't change the routine body, if you want to change it you have to drop and recreate the routine

# dropping rountines #

	DROP PROCEDURE [IF EXISTS] proc_name;
	DROP FUNCTION [IF EXISTS] func_name;

# invoking routines #

To invoke a procedure use CALL

	CALL foo();

to invoke a function, just use the same way as builtin function

	SELECT foofunc(10);

# obtaining meta info #

	SELECT * FROM INFORMATION_SCHEMA.ROUTINES
	WHERE ROUTINE_NAME = 'foo'
	AND ROUTINE_SCHEMA = 'bar'\G

	SHOW PROCEDURE STATUS;
	SHOW FUNCTION STATUS;
	SHOW PROCEDURE STATUS LIKE 'w%';

	SHOW CREATE PROCEDURE foo;
	SHOW CREATE FUNCTION bar;

# privileges #

 - for create a routine you need CREATE ROUTINE privileges
 - for invokine a rountine you need EXECUTE privileges for it
 - to drop or alter a routine you must have ALTER ROUTINE privileges for it
 - to grant privileges for a routine, you must have the GRANT OPTION privileges for it  

to grant GRANT option you have to explicitly write it

	GRANT ALL ON foo.* to 'bar'@'localhost' WITH GRANT OPTION;

	GRANT EXECUTE, ALTER ROUTINE ON PROCEDURE foo.foo TO 'bar'@'localhost';