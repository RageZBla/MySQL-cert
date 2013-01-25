---
layout: post
title: Client Server concepts
---

# General #

MySQL is muti OS (Windows, Mac, Unix), multi thread, multi connections. 

MySQL comes in several form
 
 - max with all optional feature
 - vanilla 

 Server is the MySQL daemon, host is the piece of hardware running the OS and the daemon.

 Client programs: 

  - MySQL Query Browser and MySQL Administrator
  - mysql command line tool
  - other command line tools such as: mysqlimport, mysqldump, mysqlcheck

Some commands directly interact with files and not MySQL server, those are not considered client programs, i.e.: 

 - myisamchhk
 - etc... 

# mysql command tool #

the command line tool accept two form of arguments: 

 - long options, i.e. --help (case sensitive)
 - short options -h (case sensitive)

Passing values to options

 	mysql --host=foo.bar.net
 	mysql -h foo.bar.net
 	mysql -hfoo.bar.net

# connection parameters #

list of connection related arguments: 

  - --protocol: protocol to use (tcp, socket, pipe, memory)
  - --host: hostname
  - --port: TCP port 
  - --shared-memory-base-name: the shared memory name (default: MYSQL case sensitive)
  - --socket socket filename (default: /tmp/mysql.sock) or named-pide name (MySQL case insensitive)

list of identification related arguments: 

  - --user username (unix: logged user by default, windows ODBC)
  - --password password

Protocols: 

   - tcp: availabe everywhere
   - socket: unix only
   - pipe: windows only
   - memory: windows only

To use pipe protocol, the daemon must be runned with the --enable-named-pipe

Regarding, shared memory you need --shared-memory and the default connection protocol become memory. 

Specifying localhost as hostname, make mysql use the socket protocol by default mysql try to /tmp/mysql.sock. 

On windows, specifying . as hostname would use the named pipe protocols. 

To explicity connect using TCP protocol, use --protocol=tcp, or hostname 127.0.0.1 or FQDN or IP number. 

	mysql -p pass_val

**is not valid!**, you have to write

	mysql -ppass_val

-C, --compress compress the traffic between the server and the client. 

# Using option files #

Option files are organised by group

The group [client] regroup the options for all client programs(mysql, mysqldump, mysqlcheck, etc). 

the config file use the same arguments as the programs, only the long form

	[mysql]
	host = foo.bar.net
	compress

_spaces are allowed around the equal sign_

On windows MySQL looks for config file in these order

 1. c:\Windows\my.ini
 2. c:\Windows\my.cnf
 3. c:\my.ini
 4. c:\my.cnf

On Linux

 1. /etc/my.cnf
 2. ~/.my.cnf

To tell mysql to read only a single file use 

	mysql --defaults-file=~/my.cnf

To tell mysql to read an extra files
	
	mysql --defaults-extra-file=file_name

If you want mysql not reading default locations

	mysql --no-defaults

In the config file you can include other config file or directory

	!include file_name
	includedir dir_name

# default database #

	mysql foobar
	SELECT * FROM foobar.foo;
	USE foobar;

# SQL Modes #

SQL modes are not case sensitive, if you have no mode or more than one you should quote it

clear the SQL mode
	
	SET sql_mode = '';

Set some single mode

	SET sql_mode = ANSI_QUOTES;
	SET sql_mode = 'TRADITIONAL';

Set multiple modes

	SET sql_model = 'IGNORE_SPACE,ANSI_QUOTES';

show the current mode

	SELECT @@sql_mode;

Modes: 

 - ANSI_QUOTES: identifier quoting is "
 - IGNORE_SPACE: allow space between name and function i.e LENGTH (), also cause FUNCTION to be reserved words
 - ERROR_FOR_DIVISION_BY_ZERO: self explanatory, by default division by zero returns NULL
 - STRICT_TRANS_TABLES: make the transactional tables (InnoDB) stritcs
 - STRICT_ALL_TABLES: strict for everything
 - TRADITIONAL: make MySQL behave like a "traditional" RDBM
 - ANSI: make MySQL behave like an "ANSI" RDBM


