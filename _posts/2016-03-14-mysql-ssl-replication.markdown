---
layout: post
comments: true
tags:
- MySQL
- SSL
- replication
title:  "MySQL Replication with SSL"
---

<style type="text/css">
.custom-code { background-color: #ffe; }
</style>

These are the steps I used to create a MySQL replication slave using SSL, and were performed using MySQL 5.5 and Ubuntu 15.10.

The <span class="custom-code">highlighted items</span> should be replaced with your own items.
I hope someone else finds this useful!

<br>

<span>1. On the main database server, set up the replication user from the MySQL command line:</span>
<pre>
# create the user and require SSL
CREATE USER '<span class="custom-code">replication_user</span>'@'%' IDENTIFIED BY '<span class="custom-code">replication_password</span>';

# set the permissions
GRANT REPLICATION CLIENT, REPLICATION SLAVE ON *.* TO '<span class="custom-code">replication_user</span>'@'%' REQUIRE SSL;

# reload the user privileges
FLUSH PRIVILEGES;
</pre>

<br>

<span>2. On the main database server, create the SSL keys:</span>
<pre>
sudo mkdir /etc/ssl/certs/mysql
sudo chown mysql /etc/ssl/certs/mysql
cd /etc/ssl/certs/mysql
sudo openssl req -x509 -newkey rsa:2048 -keyout master-private.pem -out master-public.pem -nodes -days 3650

# convert the key to the correct format for mysql
sudo openssl rsa -in master-private.pem -out master-private.pem

sudo cp master-public.pem ca-cert.pem
sudo chown mysql:mysql *
</pre>

<br>

<span>3. On the slave database server, add the SSL keys:</span>
<pre>
sudo mkdir /etc/ssl/certs/mysql
sudo chown <span class="custom-code">SLAVE_USER</span> /etc/ssl/certs/mysql
cd /etc/ssl/certs/mysql

# transfer the certificate from the master database server to the slave database server
scp <span class="custom-code">MASTER_USER</span>@<span class="custom-code"><span class="custom-code">MASTER_IP_ADDRESS</span></span>:/etc/ssl/certs/mysql/ca-cert.pem .

# create the slave's ssl keys
sudo openssl req -x509 -newkey rsa:2048 -keyout slave-private.pem -out slave-public.pem -nodes -days 3650

# convert the key to the correct format for mysql
sudo openssl rsa -in slave-private.pem -out slave-private.pem

# append the public key to the certificate
sudo bash -c "cat slave-public.pem >> ca-cert.pem"

# change file ownerships back to mysql
sudo chown mysql:mysql /etc/ssl/certs/mysql -R
</pre>

<br>

<span>4. On the main database server, get the new certificate from the slave:</span>
<pre>
cd /etc/ssl/certs/mysql
sudo chown <span class="custom-code">MASTER_USER</span> ca-cert.pem
scp <span class="custom-code">SLAVE_USER</span>@<span class="custom-code">SLAVE_IP_ADDRESS</span>:/etc/ssl/certs/mysql/ca-cert.pem .
</pre>

Set the file permissions back:
<pre>sudo chown mysql ca-cert.pem</pre>

<br>

<span>5. On the main database server, modify the MySQL configuration:</span>
<pre>
sudo nano /etc/mysql/my.cnf
</pre>
<pre>
[mysqld]
ssl
ssl-ca=/etc/ssl/certs/mysql/ca-cert.pem
ssl-cert=/etc/ssl/certs/mysql/master-public.pem
ssl-key=/etc/ssl/certs/mysql/master-private.pem

log-bin=mysql-bin
server-id=2
expire_logs_days=30
</pre>

Restart MySQL:
<pre>sudo service mysql restart</pre>

Verify that SSL is enabled via the MySQL command line:
<pre>SHOW VARIABLES LIKE '%ssl%';</pre>

You can test the connection at this point from the slave server, using:
<pre>mysql -u<span class="custom-code">replication</span> -p -h<span class="custom-code">MASTER_IP_ADDRESS</span> --ssl-ca /etc/ssl/certs/mysql/ca-cert.pem</pre>

<br>

<span>6. On the main database server, get the log position and dump the database.</span>

From the MySQL command line (session 1):
<pre>
# prevent any writes from occurring on the database
# the web app will return errors until the lock is released
FLUSH TABLES WITH READ LOCK;
</pre>

From another MySQL command line (session 2):
<pre>
# write down the output from this command
SHOW MASTER STATUS;
</pre>

<pre>
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| <span class="custom-code">mysql-bin.000055</span> |   <span class="custom-code">555555</span> |              |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.00 sec)
</pre>

Close session 2, and dump the database:
<pre>
cd ~/backups
mysqldump -uroot -p --databases <span class="custom-code">DATABASE_NAME</span> | gzip > <span class="custom-code">20151231</span>.sql.gz
</pre>

Back to session 1:
<pre>
# release the lock that is preventing writes to the database
UNLOCK TABLES;
</pre>

<br>

<span>7. On the slave database server, allow AppArmor to access the ca-cert.pem file:</span>
<pre>
sudo nano /etc/apparmor.d/usr.sbin.mysqld
</pre>

Add the line:
<pre>
/etc/ssl/certs/mysql/*.pem r,
</pre>

<br>

<span>8. On the slave database server, configure MySQL:</span>
<pre>
sudo nano /etc/mysql/my.cnf

[mysqld]
ssl
ssl-ca=/etc/ssl/certs/mysql/ca-cert.pem
ssl-cert=/etc/ssl/certs/mysql/slave-public.pem
ssl-key=/etc/ssl/certs/mysql/slave-private.pem

server-id=3
relay-log=mysql-relay-bin.log
log-bin=mysql-bin

# also log updates to the slave server (optional)
log_slave_updates=true
</pre>

Restart MySQL:
<pre>sudo service mysql restart</pre>

Verify that SSL is enabled via the MySQL command line:
<pre>SHOW VARIABLES LIKE '%ssl%';</pre>

<br>

<span>9. On the slave database server, import the database:</span>
<pre>
cd ~/backups
scp <span class="custom-code">MASTER_USER</span>@<span class="custom-code">MASTER_IP_ADDRESS</span>:/home/<span class="custom-code">SLAVE_USER</span>/backups/<span class="custom-code">20151231</span>.sql.gz .
gunzip <span class="custom-code">20151231</span>.sql.gz
mysql -uroot -p < <span class="custom-code">20151231</span>.sql
</pre>

<br>

<span>10. On the slave database, start replication from the MySQL command line:</span>
<pre>
# set the master credentials
CHANGE MASTER TO MASTER_HOST='<span class="custom-code">MASTER_IP_ADDRESS</span>', MASTER_USER='<span class="custom-code">replication</span>', MASTER_PASSWORD='<span class="custom-code">replication_password</span>', MASTER_SSL=1, MASTER_SSL_CA='/etc/ssl/certs/mysql/ca-cert.pem', MASTER_LOG_FILE='<span class="custom-code">mysql-bin.000055</span>', MASTER_LOG_POS=<span class="custom-code">555555</span>;

# start the slave
START SLAVE;

# check the status
SHOW SLAVE STATUS\G
</pre>
