---
layout: post
title: Data science tech support episode 3
tags: PostgreSQL psql insecure password
---

Using the PostgreSQL command line psql client?  Want to avoid typing the database user's database password for a particular database?  (And you aren't bothered by putting it in a plain text file.)  


Create the [.pgpass](https://www.postgresql.org/docs/current/static/libpq-pgpass.html) file with a [here document](http://tldp.org/LDP/abs/html/here-docs.html) and set the permissions with the [chmod](https://www.gnu.org/software/coreutils/manual/html_node/chmod-invocation.html#chmod-invocation) command. Please substitute the actual name of your database user for dbuser and the actual password for dbpass here.
```
cat > ~/.pgpass <<EOF
#
# used by psql client
# ~/.pgpass
# should be chmod 600 ~/.pgpass
#
# (alternative) location of this file in env PGPASSFILE
#
# hostname:port:database:username:password
#
*:5432:*:dbuser:dbuserpass
EOF
chmod 600 ~/.pgpass
```

Then test it. For example here I am testing using TCP/IP connection to localhost (which is actually being forwarded over ssh) for the user ec2-user to the database baseballdata.  You should not be prompted for a password:
```
 psql -U ec2-user -h localhost -d baseballdata
psql (9.5.7)
Type "help" for help.

baseballdata=# \q
```
