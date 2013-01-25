---
layout: post
title: Identifiers
---

# Syntax #

 - Identifier may contains all alphanumeric, _ underscore, dollar $
 - may begins by any legal characters
 - cannot consist entirely of digits

Identifer may be quoted using ` (or " in ANSI_QUOTES sql_mode), quoting as the following effects: 
 - spaces are allowed
 - can contain any characters exept
 	- 255 and 0 ordinal ascii
 	- . or / or \
 - may consist entirely of digits

An alias identifier can include any character, should be quoted if reserved words, alias can use ', `, "

# case sensitivity #

 - tables and databases: depends on 
   - OS, filesystem
   - lower_case_table_names is set to 1 or 2
   - should be written in same case in query
   - case sensitive
 - colums, index, stored routine, trigger identifiers as case insensitive
 - columns alias as case insensitive

# Qualified names #

	SELECT Name FROM Country;
	SELECT Country.Name FROM Country;
	SELECT world.Country.Name FROM world.Country;

Reserved words are not case sensitive