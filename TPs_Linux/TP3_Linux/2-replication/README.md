# Module 2 : Réplication de base de données

Il y a plein de façons de mettre en place de la réplication de base données de type MySQL (comme MariaDB).

MariaDB possède un mécanisme de réplication natif qui peut très bien faire l'affaire pour faire des tests comme les nôtres.

Une réplication simple est une configuration de type "master-slave". Un des deux serveurs est le *master* l'autre est un *slave*.

Le *master* est celui qui reçoit les requêtes SQL (des applications comme NextCloud) et qui les traite.

Le *slave* ne fait que répliquer les donneés que le *master* possède.

La [doc officielle de MariaDB](https://mariadb.com/kb/en/setting-up-replication/) ou encore [cet article cool](https://cloudinfrastructureservices.co.uk/setup-mariadb-replication/) expose de façon simple comment mettre en place une telle config.

Pour ce module, vous aurez besoin d'un deuxième serveur de base de données.

## Installation :
```
[audran@db2 ~]$ sudo dnf install mariadb-server
[audran@db2 ~]$ sudo systemctl start mariadb
[audran@db2 ~]$ sudo systemctl enable mariadb
[audran@db2 ~]$ sudo mysql_secure_installation
Thanks for using MariaDB!

[audran@db2 ~]$ mysqladmin -u root -p version
Enter password:
mysqladmin  Ver 9.1 Distrib 10.5.16-MariaDB, for Linux on x86_64
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Server version          10.5.16-MariaDB
Protocol version        10
Connection              Localhost via UNIX socket
UNIX socket             /var/lib/mysql/mysql.sock
Uptime:                 3 min 40 sec

Threads: 1  Questions: 19  Slow queries: 0  Opens: 20  Open tables: 13  Queries per second avg: 0.086
```

## Configuration DB:
```
[audran@db ~]$ sudo vi /etc/my.cnf

# This group is read both both by the client and the server
# use it for options that affect everything
#
[client-server]

#
# include all files from the config directory
#
!includedir /etc/my.cnf.d
[mysqld]
bind-address=10.102.1.12
server-id=1
log_bin=mysql-bin
binlog-format=ROW
```

```
[audran@db2 ~]$ sudo mysql -u root -p
MariaDB [(none)]> CREATE USER 'replication'@'%' IDENTIFIED BY 'azerty';
Query OK, 0 rows affected (0.003 sec)

MariaDB [(none)]> GRANT REPLICATION SLAVE ON *.* TO 'replication'@'%';
Query OK, 0 rows affected (0.001 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.000 sec)

MariaDB [(none)]> SHOW MASTER STATUS;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000001 |    10486 |              |                  |
+------------------+----------+--------------+------------------+
1 row in set (0.001 sec)
```
```
[audran@db ~]$ sudo mysqldump -u root nextcloud > toto.sql
[audran@db ~]$ scp toto.sql audran@10.102.1.14:/tmp/
```

## Configuration DB2 (réplication) :
```
[audran@db2 ~]$ sudo vi /etc/my.cnf

# This group is read both both by the client and the server
# use it for options that affect everything
#
[client-server]

#
# include all files from the config directory
#
!includedir /etc/my.cnf.d
[mysqld]
bind-address=10.102.1.14
server-id=2
binlog-format=ROW
```
```
[audran@db2 ~]$ sudo systemctl restart mariadb
```
```
[audran@db2 ~]$ sudo mysql -u root -p

MariaDB [(none)]> STOP SLAVE;
Query OK, 0 rows affected, 1 warning (0.000 sec)

MariaDB [(none)]> CHANGE MASTER TO MASTER_HOST = '10.102.1.12', MASTER_USER = 'replication', MASTER_PASSWORD = 'azerty',
 MASTER_LOG_FILE = 'mysql-bin.000001', MASTER_LOG_POS = 10486;
Query OK, 0 rows affected (0.005 sec)

MariaDB [(none)]> EXIT
```
```
[audran@db2 ~]$ sudo mv  /tmp/toto.sql  .
[audran@db2 ~]$ sudo vim toto.sql

CREATE DATABASE IF NOT EXISTS nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
use nextcloud;

[audran@db2 ~]$ sudo mysql -u root < toto.sql 
```

```
[audran@db2 ~]$ sudo mysql -u root
MariaDB [(none)]> START SLAVE;
Query OK, 0 rows affected, 1 warning (0.000 sec)
```

```
MariaDB [(none)]> show slave status \G;
*************************** 1. row ***************************
                Slave_IO_State: Waiting for master to send event
                   Master_Host: 10.102.1.12
                   Master_User: replication
                   Master_Port: 3306
                 Connect_Retry: 60
               Master_Log_File: mysql-bin.000001
           Read_Master_Log_Pos: 244624
                Relay_Log_File: mariadb-relay-bin.000003
                 Relay_Log_Pos: 555
         Relay_Master_Log_File: mysql-bin.000001
              Slave_IO_Running: Yes
             Slave_SQL_Running: Yes
               Replicate_Do_DB:
           Replicate_Ignore_DB:
            Replicate_Do_Table:
        Replicate_Ignore_Table:
       Replicate_Wild_Do_Table:
   Replicate_Wild_Ignore_Table:
                    Last_Errno: 0
                    Last_Error:
                  Skip_Counter: 0
           Exec_Master_Log_Pos: 244624
               Relay_Log_Space: 232017
               Until_Condition: None
                Until_Log_File:
                 Until_Log_Pos: 0
            Master_SSL_Allowed: No
            Master_SSL_CA_File:
            Master_SSL_CA_Path:
               Master_SSL_Cert:
             Master_SSL_Cipher:
                Master_SSL_Key:
         Seconds_Behind_Master: 0
 Master_SSL_Verify_Server_Cert: No
                 Last_IO_Errno: 0
                 Last_IO_Error:
                Last_SQL_Errno: 0
                Last_SQL_Error:
   Replicate_Ignore_Server_Ids:
              Master_Server_Id: 1
                Master_SSL_Crl:
            Master_SSL_Crlpath:
                    Using_Gtid: No
                   Gtid_IO_Pos:
       Replicate_Do_Domain_Ids:
   Replicate_Ignore_Domain_Ids:
                 Parallel_Mode: optimistic
                     SQL_Delay: 0
           SQL_Remaining_Delay: NULL
       Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
              Slave_DDL_Groups: 1
Slave_Non_Transactional_Groups: 0
    Slave_Transactional_Groups: 29
1 row in set (0.000 sec)
```
## TEST :
Création de table dans db :
```
[audran@db ~]$ sudo mysql -u root -p
MariaDB [(none)]> use nextcloud;
MariaDB [nextcloud]> create table users (age int);
Query OK, 0 rows affected (0.004 sec)
MariaDB [nextcloud]> show tables;

+-----------------------------+
| Tables_in_nextcloud         |
+-----------------------------+
[...]
| oc_vcategory_to_object      |
| oc_webauthn                 |
| oc_whats_new                |
| users                       |
+-----------------------------+
125 rows in set (0.000 sec)
```
<br/>

Vérification sur db2 :
```
[audran@db2 ~]$ sudo mysql -u root -p
MariaDB [(none)]> show databases;

+--------------------+
| Database           |
+--------------------+
| information_schema |
| maxime             |
| mysql              |
| nextcloud          |
| performance_schema |
+--------------------+
5 rows in set (0.000 sec)

MariaDB [(none)]> use nextcloud;
MariaDB [nextcloud]> show tables;

+-----------------------------+
| Tables_in_nextcloud         |
+-----------------------------+
[...]
| oc_webauthn                 |
| oc_whats_new                |
| users                       |
+-----------------------------+
125 rows in set (0.000 sec)
```

<br/>

✨ **Bonus** : Faire en sorte que l'utilisateur créé en base de données ne soit utilisable que depuis l'autre serveur de base de données

- inspirez-vous de la création d'utilisateur avec `CREATE USER` effectuée dans le TP2

✨ **Bonus** : Mettre en place un setup *master-master* où les deux serveurs sont répliqués en temps réel, mais les deux sont capables de traiter les requêtes.

![Replication](../pics/replication.jpg)
