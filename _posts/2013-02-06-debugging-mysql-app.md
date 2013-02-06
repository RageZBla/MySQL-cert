---
layout: post
title: Debugging MySQL applications
---

# interpreting error messages #

	SELECT * from no_luck_here;
	ERROR 1146 (42S02): Table 'mysql_cert.no_luck' doesn't exist
	ERROR MySQL_error (SQLSTATE error): description

if it is an OS error

	ERROR 1 (HY000): Can't write to file (ErrCode: 13)
	ERROR 1 (HY000): Can't write to file (ErrCode: OS specific error code)

# show warning #

	SHOW WARNINGS\G
	SHOW WARNINGS LIMIT 1,2\G
	SHOW COUNT(*) WARNINGS;

levels of severity:

 - Error: prevented the server for completing the query
 - Warning: the query completed but with some problems
 - Note: just some piece of advices

# show errors #

	SHOW ERRORS;
	SHOW ERRORS LIMIT 1,2;
	SHOW COUNT(*) ERRORS;

Only show message with Error severity!

# perror #

MySQL ships with the perror utility, this utility show textual error messages for a given OS error code

	perror 13
	Error Code 13: Access denied
	