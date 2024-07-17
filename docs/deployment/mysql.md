---
title: Installing & Configuring MySQL
---

# Installing & Configuring MySQL
To install MySQL, use the following procedure. 

## Procesures
* Install the MySQL Server 
* Created the STIG Manager MySQL Database.
* Set the suggested DB configuration options in a MySQ.
* Change the default MySQL data directory.
* Set SELinux security context for the new data directory

## Install the MySQL Server 
Install MySQL server packages: 
```sh
sudo dnf install mysql-server
```

Start the mysqld service:
```sh
systemctl start mysqld.service
``` 

Enable the mysqld service to start at boot:  
```sh
sudo systemctl enable mysqld.service
```

!!! warning
    Recommended: To improve security when installing MySQL, run the following command: 
``` bash
mysql_secure_installation
```
The command launches a fully interactive script, which prompts for each step in the process. 
The script enables you to improve security in the following ways:

* Setting a password for root accounts
* Removing anonymous users
* Disallowing remote root logins (outside the local host) 


----
## Create MySQL and STIG Manager user
Reference: [3.2.1. Configure MySQL](https://stig-manager.readthedocs.io/en/latest/installation-and-setup/db.html#mysql)

The STIG Manager API requires a dedicated MySQL database (equivalent to
a schema in other RDBMS products). The API connects to MySQL with an
account that must have a full grant to the dedicated database but does
not require server administration privileges. On first bootstrap, all
database tables, views, and static data will be created. Example
commands to prepare MySQL for initial API execution:

!!! note
    All database commands has end with a semi-colon `;` after loging in and exiting.

Login to the database: 
```sh
mysql -u root -p
```
Create database: 
```sql
CREATE DATABASE stigman;
```

Create API user account. Replace `new_password` with ypur password
```sql
CREATE USER 'stigman'@'%' IDENTIFIED BY 'new_password';
```

Grant API user account all privileges on created database
```sql
GRANT ALL ON stigman.* TO 'stigman';
```

Run `SHOW DATABASES` to lists the databases on the MySQL server host.
```sql
SHOW DATABASES
```

```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| stigman            |
| sys                |
| tecmint            |
+--------------------+
6 rows in set (0.07 sec)
```

Execute the below commands to exit mysql.
```sql
exit
```

!!! info
    Rather than using SET PASSWORD to assign passwords, ALTER USER is the preferred statement for account alterations, 
    including assigning passwords. For example:

```sql
ALTER USER 'stigman'@'%' IDENTIFIED BY 'Password123!';
```

---
## Configure MySQL for STIG Manager
Reference: [3.2.1. Configure MySQL](https://stig-manager.readthedocs.io/en/latest/installation-and-setup/db.html#mysql)
!!! warning
    Suggested DB configuration options: 

    - `sort_buffer_size` - set to at least 2M (2097152), 
    and perhaps up to 64M (Increasing the sort_buffer_size from the default of 256k may only be 
    required if you have very large detail/comment text fields). 

    - `innodb_buffer_pool_size` - set 
    to at least 256M (268435456), and perhaps up to 2GB (2147483648)

Login into mysql as the root user, enter password, press `enter`.
```sh
mysql -u root -p
```

Execute the below commands to and check the default values
```sql
SHOW VARIABLES LIKE 'sort_buffer_size';
```
```sh title="sort_buffer_size"
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| sort_buffer_size | 262144 |
+------------------+--------+
1 row in set (0.74 sec)
```
```sql
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';
```
```sh title="innodb_buffer_pool_size"
+-------------------------+-----------+
| Variable_name           | Value     |
+-------------------------+-----------+
| innodb_buffer_pool_size | 134217728 |
+-------------------------+-----------+
1 row in set (0.01 sec)
```

Edit the config file and add the details in `/etc/my.cnf`.
```sh
sudo nano /etc/my.cnf
```
Within an option file, those variables are set like this:

```sh
[mysqld]
sort_buffer_size=64M
innodb_buffer_pool_size=2G
```

```sh title="Example of /etc/my.cnf"
#
# This group is read both both by the client and the server
# use it for options that affect everything
#
[client-server]


[mysqld]
sort_buffer_size=64M
innodb_buffer_pool_size=2G

#
# include all files from the config directory
#
!includedir /etc/my.cnf.d
```


Save and close the file when done, then restart MySQL
```sh
sudo systemctl restart mysqld.service
```
Perform the instauctions in `Step 2` to validate the values have changed.
```sh title="sort_buffer_size"
+------------------+----------+
| Variable_name    | Value    |
+------------------+----------+
| sort_buffer_size | 67108864 |
+------------------+----------+
1 row in set (0.01 sec)
```
```sh title="innodb_buffer_pool_size"
+-------------------------+------------+
| Variable_name           | Value      |
+-------------------------+------------+
| innodb_buffer_pool_size | 2147483648 |
+-------------------------+------------+
1 row in set (0.00 sec)
```

---
## Changing the default MySQL Data Directory

!!! danger
    Changing the Default MySQL Data Directory in Linux has risk. Do a complete backup before
    performing this step.

Based on the expected use of the STIG Manager database server, we may want to change the 
default data directory (`/var/lib/mysql`) to a different location. This directory is expected to 
grow due to high usage. 

!!! warning
    GSC recomends changing the default for the following reasons:

    The filesystem where `/var` is stored may collapse at one point causing the entire system to fail. 

    Changing the default directory to a dedicated local/network share that we want to use to store our actual data.

For these reason we will change the default MySQL data directory to a different path.

!!! note
    We are going to assume that our new data directory is `/data/mysql`. It is important to note
    that this directory should be owned by `mysql:mysql`.


Create the new directory if it does not exist by running the folloing command:
```sh
sudo mkdir -p /data/mysql 
sudo chown -R mysql:mysql /data/mysql
```

!!! note
    To begin, it is worthy and well to identify the current data directory using the following command. 
    Do not just assume it is still `/var/lib/mysql` since it could have been changed in the past.

Execute the below commands to dentifying the Current MySQL Data Directory by 
```sh
sudo mysql -u root -p -e "SELECT @@datadir;"
```
After you enter the MySQL password, the output should be similar to.
```sh title="Current MySQL Data Directory"
+-----------------+
| @@datadir       |
+-----------------+
| /var/lib/mysql/ |
+-----------------+
```

To avoid data corruption, stop the service if it is currently running before proceeding. Use the systemd well-known commands to do so.

Execute the below commands to copy MySQL Data Directory to a new location:
```sh
sudo systemctl stop mysqld.service
sudo systemctl is-active mysqld
```
!!! info
    If the service has been brought down, the output of the last command should be as follows:

```sh title="Identify MySQL Data Directory"
[root@localhost ~]# systemctl is-active mysqld.service
inactive
[root@localhost ~]#
```

Then copy recursively the contents of `/var/lib/mysql` to `/data/mysql` preserving original permissions and timestamps:

```sh
sudo cp -R -p /var/lib/mysql/* /data/mysql
```
or
```
sudo rsync -av /var/lib/mysql/* /data/mysql 
```
Edit the configuration file (my.cnf) to indicate the new data directory (`/data/mysql in this case`).

Execute the below commands to configure the New MySQL data directory:
```sh
sudo nano /etc/my.cnf
```

Locate or add the `[client]` sections and make the following changes after the `[client-server]` block:

```sh
[client]
port=3306
socket=/data/mysql/mysql.sock
```
Save the changes and then proceed with the next step.

```sh title="Configure New MySQL Data Directory"
#
# This group is read both both by the client and the server
# use it for options that affect everything
#
[client-server]


[mysqld]
sort_buffer_size=64M
innodb_buffer_pool_size=2G


[client]
port=3306
socket=/data/mysql/mysql.sock


#
# include all files from the config directory
#
!includedir /etc/my.cnf.d
```

Now lets change the server settings in `/etc/my.cnf.d/mysql-server.cnf`
```sh
sudo nano /etc/my.cnf.d/mysql-server.cnf
```

Make the following changes then save and exit the editor:

* `datadir=/var/lib/mysql` -> `/data/mysql`
* `socket=/var/lib/mysql/mysql.sock` -> `/data/mysql/mysql.sock`

```sh
#
# This group are read by MySQL server.
# Use it for options that only the server (but not clients) should see
#
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/en/server-configuration-defaults.html

# Settings user and group are ignored when systemd is used.
# If you need to run mysqld under a different user or group,
# customize your systemd unit file for mysqld according to the
# instructions in http://fedoraproject.org/wiki/Systemd

[mysqld]
datadir=/data/mysql
socket=/data/mysql/mysql.sock
log-error=/var/log/mysql/mysqld.log
pid-file=/run/mysqld/mysqld.pid
```

---
## Set SELinux Security Context to Data Directory

!!! note
    This step is only applicable to **RHEL/CentOS** and its derivatives.

Add the SELinux security context to `/data/mysql` before restarting MySQL by running the following commands:

```sh
sudo semanage fcontext -a -t mysqld_db_t "/data/mysql(/.*)?"
sudo restorecon -R /data/mysql
```
Next restart the MySQL service then check the status.

```sh title="MySQL commands"
sudo systemctl start mysqld.service
sudo systemctl is-active mysqld
```

Now, use the same command as in `Step 1` to verify the location of the new data directory:
```sh title="Verify MySQL New Data Directory"
sudo mysql -u root -p -e "SELECT @@datadir;"
```
```sh title="New MySQL Data Directory"
+-----------------+
| @@datadir       |
+-----------------+
| /data/mysql/    |
+-----------------+
```

---
## (Optional) Create a Database to Confirm Data Directory

Run the following command to login to MySQL and create a new database in the new directory `/data/mysql`:

```sh title="Check MySQL New Data Directory"
mysql -u root -p -e "CREATE DATABASE gsctest;"

```
If you did not receive an error, Execute the below commands to remove the new Database you just created:

```sh title="Drop MySQL gsctest Data Base"
mysql -u root -p -e "DROP DATABASE gsctest;"

```
Both commands are expected to return an "ok" message.

---
## Extra
Extra configuration/deployment scenarios can be found in `Extra - MySQL` section


## Summary
!!! success annotate "Summary"
    In these procedures, on a RHEL 7/8 distributions, you have:

    * Install the MySQL Server 
    * Created the STIG Manager MySQL Database.
    * Set the suggested DB configuration options in a MySQ.
    * Change the default MySQL data directory.
    * Set SELinux security context for the new data directory

## Next Step
[Installing & Configuring Keycloak](keycloak.md)