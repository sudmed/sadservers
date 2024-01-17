# "Bucharest": Connecting to Postgres

**Description:** A web application relies on the PostgreSQL 13 database present on this server. However, the connection to the database is not working. Your task is to identify and resolve the issue causing this connection failure. The application connects to a database named `app1` with the user `app1user` and the password `app1user`.  

**Test:** Running `PGPASSWORD=app1user psql -h 127.0.0.1 -d app1 -U app1user -c '\q'` succeeds (does not return an error).  

---

### Solution:
#### 1. Reconnaissance on the server
`ls -la`  
```console
total 36
drwxr-xr-x 5 admin admin 4096 Nov 25 18:05 .
drwxr-xr-x 3 root  root  4096 Nov 22 01:04 ..
drwx------ 3 admin admin 4096 Nov 22 01:06 .ansible
-rw-r--r-- 1 admin admin  220 Mar 27  2022 .bash_logout
-rw-r--r-- 1 admin admin 3526 Mar 27  2022 .bashrc
-rw-r--r-- 1 admin admin  807 Mar 27  2022 .profile
drwx------ 2 admin admin 4096 Nov 22 01:04 .ssh
-rw-r--r-- 1 admin admin  829 Nov 25 18:05 README.txt
drwxr-xr-x 2 admin root  4096 Nov 25 17:29 agent
```

`cd agent && ls -la`  
```console
total 11144
drwxr-xr-x 2 admin root      4096 Nov 25 17:29 .
drwxr-xr-x 5 admin admin     4096 Nov 25 18:05 ..
-rwxr-xr-x 1 admin admin      319 Nov 25 17:29 check.sh
-rwxr-xr-x 1 admin root  11397096 Nov 22 01:07 sadagent
-rw-r--r-- 1 admin admin        0 Nov 22 01:07 sadagent.txt
```

`cat check.sh`  
```console
#!/bin/bash
# # DO NOT MODIFY THIS FILE ("Check My Solution" will fail)

# Set the password and try to connect to the postgres server
PGPASSWORD=app1user psql -h 127.0.0.1 -d app1 -U app1user -c '\q' 2>/dev/null
STATUS=$?

# Check the exit status
if [ ${STATUS} -eq 0 ]; then
    echo -n "OK"
else
    echo -n "NO"
fi
```

`pg_lsclusters`  
```console
Ver Cluster Port Status Owner    Data directory              Log file
13  main    5432 online postgres /var/lib/postgresql/13/main /var/log/postgresql/postgresql-13-main.log
```

`PGPASSWORD=app1user psql -h 127.0.0.1 -d app1 -U app1user -c '\q'`  
```console
psql: error: FATAL:  pg_hba.conf rejects connection for host "127.0.0.1", user "app1user", database "app1", SSL on
FATAL:  pg_hba.conf rejects connection for host "127.0.0.1", user "app1user", database "app1", SSL off
```

`cat /etc/postgresql/13/main/postgresql.conf | grep pg_hba.conf`  
```console
hba_file = '/etc/postgresql/13/main/pg_hba.conf'        # host-based authentication file
```

`cat /etc/postgresql/13/main/pg_hba.conf`  
<details>

  <summary>output</summary>

```bash
# PostgreSQL Client Authentication Configuration File
# ===================================================
#
# Refer to the "Client Authentication" section in the PostgreSQL
# documentation for a complete description of this file.  A short
# synopsis follows.
#
# This file controls: which hosts are allowed to connect, how clients
# are authenticated, which PostgreSQL user names they can use, which
# databases they can access.  Records take one of these forms:
#
# local         DATABASE  USER  METHOD  [OPTIONS]
# host          DATABASE  USER  ADDRESS  METHOD  [OPTIONS]
# hostssl       DATABASE  USER  ADDRESS  METHOD  [OPTIONS]
# hostnossl     DATABASE  USER  ADDRESS  METHOD  [OPTIONS]
# hostgssenc    DATABASE  USER  ADDRESS  METHOD  [OPTIONS]
# hostnogssenc  DATABASE  USER  ADDRESS  METHOD  [OPTIONS]
#
# (The uppercase items must be replaced by actual values.)
#
# The first field is the connection type: "local" is a Unix-domain
# socket, "host" is either a plain or SSL-encrypted TCP/IP socket,
# "hostssl" is an SSL-encrypted TCP/IP socket, and "hostnossl" is a
# non-SSL TCP/IP socket.  Similarly, "hostgssenc" uses a
# GSSAPI-encrypted TCP/IP socket, while "hostnogssenc" uses a
# non-GSSAPI socket.
#
# DATABASE can be "all", "sameuser", "samerole", "replication", a
# database name, or a comma-separated list thereof. The "all"
# keyword does not match "replication". Access to replication
# must be enabled in a separate record (see example below).
#
# USER can be "all", a user name, a group name prefixed with "+", or a
# comma-separated list thereof.  In both the DATABASE and USER fields
# you can also write a file name prefixed with "@" to include names
# from a separate file.
#
# ADDRESS specifies the set of hosts the record matches.  It can be a
# host name, or it is made up of an IP address and a CIDR mask that is
# an integer (between 0 and 32 (IPv4) or 128 (IPv6) inclusive) that
# specifies the number of significant bits in the mask.  A host name
# that starts with a dot (.) matches a suffix of the actual host name.
# Alternatively, you can write an IP address and netmask in separate
# columns to specify the set of hosts.  Instead of a CIDR-address, you
# can write "samehost" to match any of the server's own IP addresses,
# or "samenet" to match any address in any subnet that the server is
# directly connected to.
#
# METHOD can be "trust", "reject", "md5", "password", "scram-sha-256",
# "gss", "sspi", "ident", "peer", "pam", "ldap", "radius" or "cert".
# Note that "password" sends passwords in clear text; "md5" or
# "scram-sha-256" are preferred since they send encrypted passwords.
#
# OPTIONS are a set of options for the authentication in the format
# NAME=VALUE.  The available options depend on the different
# authentication methods -- refer to the "Client Authentication"
# section in the documentation for a list of which options are
# available for which authentication methods.
#
# Database and user names containing spaces, commas, quotes and other
# special characters must be quoted.  Quoting one of the keywords
# "all", "sameuser", "samerole" or "replication" makes the name lose
# its special character, and just match a database or username with
# that name.
#
# This file is read on server startup and when the server receives a
# SIGHUP signal.  If you edit the file on a running system, you have to
# SIGHUP the server for the changes to take effect, run "pg_ctl reload",
# or execute "SELECT pg_reload_conf()".
#
# Put your actual configuration here
# ----------------------------------
#
# If you want to allow non-local connections, you need to add more
# "host" records.  In that case you will also need to make PostgreSQL
# listen on a non-local interface via the listen_addresses
# configuration parameter, or via the -i or -h command line switches.


# DO NOT DISABLE!
# If you change this first entry you will need to make sure that the
# database superuser can access the database using some other method.
# Noninteractive access to all databases is required during automatic
# maintenance (custom daily cronjobs, replication, and similar tasks).
#
# Database administrative login by Unix domain socket
local   all             postgres                                peer
host    all             all             all                     reject
host    all             all             all                     reject

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            md5
host    replication     all             ::1/128                 md5
```

</details>


#### 2. Problem seems like connection method is `reject` (even twice)
`cat /etc/postgresql/13/main/pg_hba.conf`  
```console
<...>
# Database administrative login by Unix domain socket
local   all             postgres                                peer
host    all             all             all                     reject
host    all             all             all                     reject
<...>
```


#### 3. Change connection method to `md5`
`vi /etc/postgresql/13/main/pg_hba.conf`  
```console
# Database administrative login by Unix domain socket
local   all             postgres                                peer
host    all             all             all                     reject
host    all             all             all                     reject
```

`systemctl restart postgresql`  


#### 5. Validate the task
`PGPASSWORD=app1user psql -h 127.0.0.1 -d app1 -U app1user -c '\q'`  
```console
$
```

`cd ~/agent && ./check.sh`  
```console
OK
```
