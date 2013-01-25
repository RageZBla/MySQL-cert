---
layout: post
title: MySQL connectors
---

# libmysqlclient #

mysql client depends on a library to handle the communication with server: libmysqlclient. 




# Mysql ODBC #

Use the C library, act as adapter for ODBC standard. Available on Windows and Linux

Handles such protocols: 

 - TCP
 - socket
 - named pipes



# MySQL Connector/J #

JDBC connector, JDBC version 3.0 , Type 4 pure java. 

Handle server side prepared statements, stored rountine and unicode

Windows and Unix

*does not rely on C lib*

allows such protocols :
 - TCP protocol
 - named pipe

# MySQL connector/NET #

Implementation of the protocols in C#, supports ADO.net. 

Allow all protocols
 - TCP
 - named pipes
 - shared memory
 - unix socket (using Mono)

 Support advanced features, prepared statements, stored routines, unicode. 

 