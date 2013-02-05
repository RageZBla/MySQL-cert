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
 