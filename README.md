# OracleZabbixConnector
How to connect to Oracle Database and execute SQL queries from Zabbix

# This setup is running on:
1. CentOS 7
2. Zabbix 3.0.3
3. Oracle drivers 12.1

# How to:
Zabbix has an option to monitor databases while using ODBC connections. This tutorial describes the steps to install and configure ODBC on a Zabbix server.

We use the "//oracleserver.local" server and the "oracle_test" database for this tutorial. Also we use an Oracle driver to connect to the database. Of course, other types of databases are also supported.

*  Sign in to the Zabbix server en switch to root
*  Copy the files in the 'OracleDrivers' folder to your server
*  Install the RPM's while using these commands in exactly this order:

  ```
# rpm -Uvh ./oracle-instantclient12.1-basic-12.1.0.2.0-1.x86_64.rpm
```
```
# rpm -Uvh ./oracle-instantclient12.1-odbc-12.1.0.2.0-1.x86_64.rpm
```

* Edit the file "/etc/odbcinst.ini" and add the "OracleODBC-12.1" section as shown in this working configuration

```
# Example driver definitions

[OracleODBC-12.1]
Description=Oracle ODBC driver for Oracle
Driver=/usr/lib/oracle/12.1/client64/lib/libsqora.so.12.1
FileUsage=1
Driver Logging=7
UsageCount=2

# Driver from the postgresql-odbc package
# Setup from the unixODBC package
[PostgreSQL]
Description     = ODBC for PostgreSQL
Driver          = /usr/lib/psqlodbcw.so
Setup           = /usr/lib/libodbcpsqlS.so
Driver64        = /usr/lib64/psqlodbcw.so
Setup64         = /usr/lib64/libodbcpsqlS.so
FileUsage       = 1


# Driver from the mysql-connector-odbc package
# Setup from the unixODBC package
[MySQL]
Description     = ODBC for MySQL
Driver          = /usr/lib/libmyodbc5.so
Setup           = /usr/lib/libodbcmyS.so
Driver64        = /usr/lib64/libmyodbc5.so
Setup64         = /usr/lib64/libodbcmyS.so
FileUsage       = 1
```

* Create the file "/etc/odbc.ini" and add this:

```
[oracle_zabbix]
Description=Oracle Test Database
Driver=OracleODBC-12.1
Trace=yes
TraceFile=/tmp/odbc_oracle.log
Database=//oracleserver.local:1525/oracle_test
UserID=
Password=
Port=1525
```
* Add the following environment variables via the command line:

```
export ORACLE_HOME=/usr/lib/oracle/12.1/client64
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$ORACLE_HOME/lib
export TWO_TASK=//oracleserver.local:1525/oracle_test
```
To make this more permanent, create a file called "oracleenv.sh" in the folder "/etc/profile.d/" and add the above mentioned lines.

* After creating the file, run this command:

```
chmod 644 /etc/profile.d/oracleenv.sh
```

* Use "iSQL" to test if the connection is working:

```
# isql oracle_zabbix
+---------------------------------------+
| Connected!                            |
|                                       |
| sql-statement                         |
| help [tablename]                      |
| quit                                  |
|                                       |
+---------------------------------------+
SQL>
```

* Now you have to add the environment variables to the Zabbix environment. Create a file: "vim /etc/sysconfig/zabbix-server" and add the following lines:

```
ORACLE_HOME=/usr/lib/oracle/12.1/client64
LD_LIBRARY_PATH=/usr/lib/oracle/12.1/client64/lib:/usr/lib64
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/lib/oracle/12.1/client64/lib
ORACLE_SID=EGELO
TWO_TASK=//oracleserver.local:1525/oracle_test

export ORACLE_HOME
export LD_LIBRARY_PATH
export PATH
export ORACLE_SID
export TWO_TASK
```

* Restart Zabbix server by using this command:

```
# systemctl restart zabbix-server.service
```

Now you are able to create an item in Zabbix with Type "Database Monitoring" and you can enter your SQL query.

For more information, have a look at the [Zabbix Documentation] https://www.zabbix.com/documentation/3.0/manual/config/items/itemtypes/odbc_checks
