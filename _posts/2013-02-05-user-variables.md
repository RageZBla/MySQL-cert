---
layout: post
title: User variables
---

Syntax:

	SET @var1 = 'foo';
	SET @var2 := 'bar';

You can use = or := to assign an user variable, you __have to use := in a select statement__.

	SELECT @var3 := 'foobar';

You can assign multiple variables at once

	SET @var1 = 'foo', @var2 = 'bar';

You refer to a variable not initialized the value is NULL. 

variable can in replacement of an expression, just it can replace constant, constant are used in: 

 - LIMIT
 - LOAD DATA INFILE fileename

User variable are used in: 

 - using EXECUTE, prepared statement
 - LOAD DATA INFILE to transform value before insert

User variable are not the same as variables declared in stored procedures. 

User variable name are not case sensitive, they are specific to the connection, when connection end all the variables are lost. 

When user variable carries a non binary string, a charset and collation is associated. 
