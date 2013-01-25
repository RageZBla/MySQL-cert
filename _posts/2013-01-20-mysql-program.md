---
layout: post
title: mysql program
---

# Interactive mode #

Statement are delimited by ";" or \g or \G, depending the state of the query mysql change the prompt: 

	- -> means it is waiting for the ";" terminator
	- '> waiting for a '
	- "> waiting for a "
	- `> waiting for a `
	- /*> waiting for end C comment

\c allows to cancel the statement

\q allows to quit

\G format results vertically, make query outputing lot of data more easy readeable. 

Execute a single query

	mysql -e "SELECT VERSION()"

using a script in interactive mode

	SOURCE script.sql
	SOURCE ../script.sql
	SOURCE /home/foobar/script.sql

The path is relative to current directory(i.e. the directory wher you mysql from)

# Output format #

	--batch or -B: output using tab
	--table or -t: output using characters to make a table
	--raw or -r: output without escaping special characters such as \n or \r
	--html or -H: output HTML
	--xml or -X: XML 
# Client command and SQL statement #

mysql program parse a mixture of client statement and SQL. The client statement interact with the program and the SQL is sent to the server. 

example of client statement

	STATUS;

Client statement (non exhaustive list): 
 - STATUS or \s: give some info about the connection to the server
 - SOURCE or \.: execute a script
 - HELP or \h: help
 - CLEAR or \c: cancel

Long form of commands cannot be entered second line of a statement. 

The short form is available everywhere. 

This behaviour can be overriden by using --named-commands. 

# Server side help #

	HELP contents;
	HELP show; 

*This require the mysql database to be loaded, usually the case*

# safe updates #

mysql as a special arguments called --safe-updates or --i-am-a-dummy. 

This mode would: 

 - block any UPDATE or DELETE statement without a WHERE clause using some key values, or without a LIMIT clause
 - limit the ouput of a SELECT to 1000 rows, unless LIMIT statement state other
 - a select query shouldn't process more than 1 millions rows, i.e. bad joins 
