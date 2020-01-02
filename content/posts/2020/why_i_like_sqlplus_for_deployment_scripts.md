---
title: Why I Like SQL*Plus for Deployment Scripts
description: SQLcl is eating my error messages
tags: [oracle, script]
lang: en
publishdate: 2020-01-02
lastmod: 2020-01-02 21:04:00
---

The following simplified example deployment masterscript is given:

```sql
set define off verify off feedback off
whenever sqlerror exit sql.sqlcode rollback
spool test.log

prompt
prompt Example Deployment Master Script
prompt ========================================

prompt Step 1: Dummy for demonstration
-- normally we call here some other scripts like
-- @other-script.sql

prompt Step 2: Error test in anonymous block
begin
  raise_application_error(-20000, 'Error test for SQL*Plus <> SQLcl comparison');
end;
/

prompt ========================================
prompt Deployment Done :-)
prompt

spool off
```

When we call this in SQL*Plus (version 19.5.0.0.0), we get the following output (in the console and also in deployment.log) with an indication of what went wrong with our deployment:

```
Example Deployment Master Script
========================================
Step 1: Dummy for demonstration
Step 2: Error test in anonymous block
begin
*
ERROR at line 1:
ORA-20000: Error test for SQL*Plus <> SQLcl comparison 
ORA-06512: at line 2 
```

Ok, this error message is stupid, but usually I get some useful information here to fix my problem in the deployment script - especially when the problematic script is a bit longer...

When we call this in SQLcl (version 19.4) we get the following output (in the console and also the deployment.log):

```
Example Deployment Master Script
========================================
Step 1: Dummy for demonstration
Step 2: Error test in anonymous block
```

Mhhh... What was going on here? Why was my script terminating in step 2?

That's one reason why I like SQL\*Plus a bit more than SQLcl for deployment scripts. The fast start time of SQL\*Plus is another reason. Nevertheless SQLcl is a cool command line tool.

Happy scripting :-)<br>
Ottmar

P.S.: There is a small [Twitter discussion][twitter] around this...


[post]: /posts/2020-01-01-download_blobs_with_sqlplus
[twitter]: https://twitter.com/ogobrecht/status/1212646721127366656

