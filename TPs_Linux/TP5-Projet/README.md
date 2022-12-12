# TP5 - Streaming musique

# I- Mise en place de Koel 
<br/>

🖥️ VM ``stream.tp5.linux`` : 10.104.1.20

<br/>

- Récupération du code :

    ```
    [audran@stream]$ mkdir project
    [audran@stream]$ cd project/
    [audran@stream project]$ sudo dnf install git
    [audran@stream project]$ git clone https://github.com/koel/koel.git .
    [audran@stream project]$ git checkout latest
    ```
<br/>

- Installation de PHP :
    - Récupération remi repo :
    ```
    [audran@stream project]$ sudo dnf update -y
    [audran@stream project]$ sudo dnf config-manager --set-enabled crb
    [audran@stream project]$ sudo dnf install epel-release
    [audran@stream project]$ sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-9.rpm

    ```

    - Installation PHP :
    ```
    [audran@stream project]$ sudo dnf module list php

    Remi's Modular repository for Enterprise Linux 9 - x86_64
    Name               Stream                    Profiles                                Summary
    php                remi-7.4                  common [d], devel, minimal              PHP scripting language
    php                remi-8.0              common [d], devel, minimal              PHP scripting language
    php                remi-8.1                  common [d], devel, minimal              PHP scripting language
    php                remi-8.2                  common [d], devel, minimal              PHP scripting language

    [audran@stream project]$ sudo dnf module enable php:remi-8.0
    [audran@stream project]$ sudo dnf install php php-zip php-intl php-mysqlnd php-dom php-simplexml php-xml php-xmlreader php-curl php-exif php-ftp php-gd php-json php-mbstring php-posix
    ```
    <br/>


- Installation de Composer :
```
[audran@stream project]$ sudo dnf install wget -y
```

```
[audran@stream project]$ php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"

[audran@stream project]$ php -r "if (hash_file('sha384', 'composer-setup.php') === '$(wget -qO - https://composer.github.io/installer.sig)') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"

[audran@stream project]$ php composer-setup.php
[audran@stream project]$ sudo mv composer.phar /usr/local/bin/composer
[audran@stream project]$ composer install
```

Unzip de Composer :
```
[audran@stream project]$ mkdir toto
[audran@stream project]$ cd toto
[audran@stream toto]$ gunzip composer
[audran@stream toto]$ mv composer composer.gz
[audran@stream toto]$ gunzip composer.gz
[audran@stream toto]$ sudo mv composer /usr/local/bin/
[audran@stream project]$ cd ..
[audran@stream project]$ rm -rf toto
```

<br/>

- Installation de Mariadb :
    - Installation :
    ```
    [audran@stream ~]$ sudo dnf install mariadb-server -y
    [audran@stream ~]$ sudo systemctl enable mariadb
    [audran@stream ~]$ sudo systemctl start mariadb
    ```

    - Ouverture du port :
    ```
    [audran@stream ~]$ sudo firewall-cmd --add-port=3306/tcp --permanent
    [audran@stream ~]$ sudo firewall-cmd --reload
    ```

    - Création de la database :
    ```
    [audran@stream project]$ sudo mysql -u root -p

    MariaDB [(none)]> CREATE USER 'audran'@'10.104.1.20' IDENTIFIED BY 'azerty';
    Query OK, 0 rows affected (0.003 sec)

    MariaDB [(none)]> CREATE DATABASE IF NOT EXISTS streamMusic CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
    Query OK, 0 rows affected (0.001 sec)

    MariaDB [(none)]> GRANT ALL PRIVILEGES ON streamMusic.* TO 'audran'@'10.104.1.20';

    MariaDB [(none)]> FLUSH PRIVILEGES;
    Query OK, 0 rows affected (0.000 sec)
    ```
<br/>

- Lancement de koel :
    - Installation de yarn :
    ```
    [audran@stream project]$ sudo dnf install npm
    [audran@stream project]$ sudo npm install --global yarn
    ```
    <br/>

    - Settings :
    ```
    [audran@stream project]$ php artisan koel:init
    ************************************
    *     KOEL INSTALLATION WIZARD     *
    ************************************

    Clearing caches ........................................................................................... 6ms DONE
    INFO  .env file exists -- skipping.

    Retrieving app key ........................................................................................ 0ms DONE

    INFO  Using app key: base64:GZxo93rni...

    WARN  Cannot connect to the database. Let's set it up.

    Your DB driver of choice [MySQL/MariaDB]:
    [mysql     ] MySQL/MariaDB
    [pgsql     ] PostgreSQL
    [sqlsrv    ] SQL Server
    [sqlite-e2e] SQLite
    > mysql

    DB host:
    > 10.104.1.20

    DB port (leave empty for default):
    >

    DB name:
    > streamMusic

    DB user:
    > audran

    DB password:
    > azerty

    Migrating database ...................................................................................... 529ms DONE
    Creating default admin account ........................................................................... 46ms DONE
    Seeding data ............................................................................................. 24ms DONE

    The absolute path to your media directory. If this is skipped (left blank) now, you can set it later via the web interface.

    Media path []:
    >

    yarn install v1.22.19
    [1/4] Resolving packages...
    [2/4] Fetching packages...

    [...]

    [OK] All done!
    ```
    <br/>

    Modification de localhost par 10.104.1.20 :

    ```
    [audran@stream project]$ vim .env

    APP_URL=http://10.104.1.20:8000
    ```

    <br/>

    Ouverture du port 8000 :
    ```
    [audran@stream project]$ sudo firewall-cmd --add-port=8000/tcp
    [audran@stream project]$ sudo firewall-cmd --add-port=8000/tcp permanent
    [audran@stream project]$ sudo firewall-cmd --add-port=8000/tcp --permanent
    ```

    <br/>

    - Lancement :
    ```
    [audran@stream project]$ php artisan serve --host  10.104.1.20

    INFO  Server running on [http://10.104.1.20:8000].

    Press Ctrl+C to stop the server
    ```

    <br/>

# II - Apache

- Installation :

```
[audran@stream project]$ sudo dnf install httpd -y
Complete!
```

<br/>

- Démarrage du service Apache :
```
[audran@stream project]$ sudo systemctl start httpd
[audran@stream project]$ sudo systemctl enable httpd
```

<br/>

- Ouverture du port :
```
[audran@stream project]$ sudo firewall-cmd --add-port=80/tcp --permanent
success
[audran@stream project]$ sudo firewall-cmd --reload
success
```

<br/>

- Création fichier de conf :
```
[audran@stream project]$ sudo nano /etc/httpd/conf.d/koel.conf

<VirtualHost *:80>
  DocumentRoot /var/www/koel/public/
  ServerName  stream.tp5.linux
  <Directory /var/www/koel/public/>
    Require all granted
    AllowOverride All
    Options FollowSymLinks MultiViews
    <IfModule mod_dav.c>
      Dav off
    </IfModule>
  </Directory>
</VirtualHost>
```

<br/>

- Déplacement du projet :
```
[audran@stream ~]$ mv project/ /var/www/
[audran@stream ~]$ cd /var/www/
[audran@stream ~]$ mv project koel
[audran@stream ~]$ sudo chown apache:apache -R koel/
```

<br/>

- Restart du service et serveur Apache joignable sur http://10.104.1.20:80 :
```
[audran@stream ~]$ sudo systemctl restart httpd
```

<br/>

# III - Reverse Proxy
<br />

🖥️ VM ``proxy.tp5.linux`` : 10.104.1.21

<br/>

- installation de nginx :
```
[audran@proxy ~]$ sudo dnf install nginx -y
[audran@proxy ~]$ sudo systemctl start nginx
[audran@proxy ~]$ sudo systemctl enable nginx
```

<br/>

- Autorisation du traffic vers Nginx :
```
[audran@proxy ~]$ sudo firewall-cmd --add-port=80/tcp --permanent
[audran@proxy ~]$ sudo firewall-cmd --reload
```

<br/>

- Configuration Nginx :
```
[audran@proxy ~]$ sudo nano /etc/nginx/conf.d/proxy.conf

server {
    # On indique le nom que client va saisir pour accéder au service
    # Pas d'erreur ici, c'est bien le nom de web, et pas de proxy qu'on veut ici !
    server_name stream.tp5.linux;

    # Port d'écoute de NGINX
    listen 80;

    location / {
        # On définit des headers HTTP pour que le proxying se passe bien
        proxy_set_header  Host $host;
        proxy_set_header  X-Real-IP $remote_addr;
        proxy_set_header  X-Forwarded-Proto https;
        proxy_set_header  X-Forwarded-Host $remote_addr;
        proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;

        # On définit la cible du proxying
        proxy_pass http://10.104.1.20:80;
    }

    # Deux sections location recommandés par la doc NextCloud
    location /.well-known/carddav {
    return 301 $scheme://$host/remote.php/dav;
    }

    location /.well-known/caldav {
    return 301 $scheme://$host/remote.php/dav;
    }
}

[audran@proxy ~]$ sudo systemctl restart nginx
```

<br/>

- Modification fichier host sur la machine hôte :
```
# For example:
#
#      102.54.94.97     rhino.acme.com          # source server
#       38.25.63.10     x.acme.com              # x client host
10.104.1.21	stream.tp5.linux
```

On peut donc ouvrir koel en tapant : http://stream.tp5.linux

<br/>

- HTTPS :

On génère une paire de clés :

```
[audran@proxy ~]$ openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout domain.key -out domain.crt
[audran@proxy ~]$ sudo mv domain.crt /etc/pki/tls/certs/
[audran@proxy ~]$ sudo mv domain.key /etc/pki/tls/private/
```

On modifie la conf de nginx :
```
[audran@proxy ~]$ sudo nano /etc/nginx/conf.d/proxy.conf

server {
    # On indique le nom que client va saisir pour accéder au service
    # Pas d'erreur ici, c'est bien le nom de web, et pas de proxy qu'on veut ici !
    server_name stream.tp5.linux;

    # Port d'écoute de NGINX
    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    ssl_certificate     /etc/pki/tls/certs/domain.crt;
    ssl_certificate_key /etc/pki/tls/private/domain.key;

    location / {
        # On définit des headers HTTP pour que le proxying se passe bien
        proxy_set_header  Host $host;
        proxy_set_header  X-Real-IP $remote_addr;
        proxy_set_header  X-Forwarded-Proto https;
        proxy_set_header  X-Forwarded-Host $remote_addr;
        proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;

        # On définit la cible du proxying
        proxy_pass http://10.104.1.20:80;
    }

    # Deux sections location recommandés par la doc NextCloud
    location /.well-known/carddav {
    return 301 $scheme://$host/remote.php/dav;
    }

    location /.well-known/caldav {
    return 301 $scheme://$host/remote.php/dav;
    }
}
server {
    listen 80 default_server;
    server_name _;
    return 301 https://$host$request_uri;
}
```

On ouvre le port 443 :

```
[audran@proxy ~]$ sudo firewall-cmd --add-port=443/tcp --permanent
[audran@proxy ~]$ sudo firewall-cmd --reload
```

<br/>

# IV - Réplication
<br/>

🖥️ VM ``replication.tp5.linux`` : 10.104.1.22

- Installation de mariadb :
    ```
    [audran@replication ~]$ sudo dnf install mariadb-server
    [audran@replication ~]$ sudo systemctl start mariadb
    [audran@replication ~]$ sudo systemctl enable mariadb
    [audran@replication ~]$ sudo mysql_secure_installation
    Thanks for using MariaDB!
    ```

- Configuration de la db :
    ```
    [audran@stream ~]$ sudo vi /etc/my.cnf

    # This group is read both both by the client and the server
    # use it for options that affect everything
    #
    [client-server]

    #
    # include all files from the config directory
    #
    !includedir /etc/my.cnf.d
    [mysqld]
    bind-address=10.104.1.20
    server-id=1
    log_bin=mysql-bin
    binlog-format=ROW

    [audran@stream ~]$ sudo systemctl restart mariadb
    ```

- Création d'un utilisateur :
    ```
    [audran@stream ~]$ sudo mysql -u root -p

    MariaDB [(none)]> CREATE USER 'replication'@'%' IDENTIFIED BY 'azerty';
    Query OK, 0 rows affected (0.071 sec)

    MariaDB [(none)]> GRANT REPLICATION SLAVE ON *.* TO 'replication'@'%';
    Query OK, 0 rows affected (0.014 sec)

    MariaDB [(none)]> FLUSH PRIVILEGES;
    Query OK, 0 rows affected (0.003 sec)

    MariaDB [(none)]> STOP SLAVE;
    Query OK, 0 rows affected, 1 warning (0.002 sec)

    MariaDB [(none)]> SHOW MASTER STATUS;
    +------------------+----------+--------------+------------------+
    | File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
    +------------------+----------+--------------+------------------+
    | mysql-bin.000002 |      792 |              |                  |
    +------------------+----------+--------------+------------------+
    1 row in set (0.001 sec)

    MariaDB [(none)]> exit
    Bye
    ```

    ```
    [audran@stream ~]$ sudo mysqldump -u root streamMusic > toto.sql
    [audran@stream ~]$ scp toto.sql audran@10.104.1.22:~/
    ```

    <br/>

- Configuration DB Réplication :
    ```
    [audran@replication ~]$ sudo vi /etc/my.cnf

    # This group is read both both by the client and the server
    # use it for options that affect everything
    #
    [client-server]

    #
    # include all files from the config directory
    #
    !includedir /etc/my.cnf.d
    [mysqld]
    bind-address=10.104.1.22
    server-id=2
    binlog-format=ROW

    [audran@replication ~]$ sudo systemctl restart mariadb
    ```

<br/>

- Création de la DB :
    ```
    [audran@replication ~]$ sudo mysql -u root -p

    MariaDB [(none)]> CREATE DATABASE IF NOT EXISTS streamMusic CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
    Query OK, 1 row affected (0.000 sec)

    MariaDB [(none)]> show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | mysql              |
    | performance_schema |
    | streamMusic        |
    +--------------------+
    4 rows in set (0.000 sec)

    [audran@replication ~]$ sudo mysql -u root -p streamMusic<toto.sql
    ```

<br/>

- Activation du slave :
    ```
    [audran@replication ~]$ sudo mysql -u root -p

    MariaDB [(none)]> STOP SLAVE;
    Query OK, 0 rows affected, 1 warning (0.000 sec)

    MariaDB [(none)]> CHANGE MASTER TO MASTER_HOST = '10.104.1.20', MASTER_USER = 'replication', MASTER_PASSWORD = 'azerty',
    MASTER_LOG_FILE = 'mysql-bin.000002', MASTER_LOG_POS = 792;
    Query OK, 0 rows affected (0.005 sec)

    MariaDB [(none)]> start slave;
    Query OK, 0 rows affected (0.002 sec)

    MariaDB [(none)]> EXIT
    ```

<br/>

- Test :
    ```
    [audran@stream ~]$ sudo mysql -u root -p

    MariaDB [(none)]> create database toto;
    Query OK, 1 row affected (0.000 sec)

    MariaDB [(none)]> show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | mysql              |
    | performance_schema |
    | streamMusic        |
    | toto               |
    +--------------------+
    5 rows in set (0.001 sec)
    ```
    ```
    [audran@replication ~]$ sudo mysql -u root -p

    MariaDB [(none)]> show databases;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | mysql              |
    | performance_schema |
    | streamMusic        |
    | toto               |
    +--------------------+
    5 rows in set (0.000 sec)
    ```

<br/>

# V - Monitoring
<br/>

- Installation :
    ```
    [audran@stream ~]$ sudo dnf install epel-release -y
    Complete!

    [audran@stream ~]$ sudo wget -O /tmp/netdata-kickstart.sh https://my-netdata.io/kickstart.sh && sh /tmp/netdata-kickstart.sh
    Successfully installed the Netdata Agent.

    [audran@stream ~]$ sudo systemctl start netdata
    [audran@stream ~]$ sudo systemctl enable netdata
    ```

<br/>

- Ouverture du port :
    ```
    [audran@stream ~]$ sudo ss -alnpt
    LISTEN         0               4096                           0.0.0.0:19999                        0.0.0.0:*
    users:(("netdata",pid=4096,fd=49))

    [audran@stream ~]$ sudo firewall-cmd --add-port=19999/tcp --permanent
    [audran@stream ~]$ sudo firewall-cmd --reload
    ```

<br/>

- Test :
Pour tester : http://10.104.1.20:19999

<br/>

- Notification sur discord :
    ```
    [audran@stream ~]$ sudo nano /etc/netdata/health_alarm_notify.conf

    # sending discord notifications

    # note: multiple recipients can be given like this:
    #                  "CHANNEL1 CHANNEL2 ..."

    # enable/disable sending discord notifications
    SEND_DISCORD="YES"

    # Create a webhook by following the official documentation -
    # https://support.discordapp.com/hc/en-us/articles/228383668-Intro-to-Webhooks
    DISCORD_WEBHOOK_URL="https://discordapp.com/api/webhooks/1044058582234181714/sGCbBWmAugHk3d98p9OM_xlwktNtLfTX3Lf_O-1Zmm>

    # if a role's recipients are not configured, a notification will be send to
    # this discord channel (empty = do not send a notification for unconfigured
    # roles):
    DEFAULT_RECIPIENT_DISCORD="général"
    ```
    ```
    [audran@stream ~]$ sudo nano /etc/netdata/health.d/cpu_usage.conf
    alarm: cpu_usage
    on: system.cpu
    lookup: average -3s percentage foreach user, system
    units: %
    every: 10s
    warn: $this > 50
    crit: $this > 80
    info: CPU utilization of users on the system itself.
    ```
    ```
    [audran@stream ~]$ sudo systemctl restart netdata
    [audran@stream ~]$ sudo dnf install stress-ng
    [audran@stream ~]$ stress-ng --vm 2 --vm-bytes 1G --timeout 60s
    ```

<br/>

# VI - Fail2Ban
<br/>

```
[audran@stream ~]$ sudo dnf install fail2ban -y
[audran@stream ~]$ sudo systemctl start fail2ban.service
[audran@stream ~]$ sudo systemctl enable fail2ban.service
Created symlink /etc/systemd/system/multi-user.target.wants/fail2ban.service → /usr/lib/systemd/system/fail2ban.service.
```

```
[audran@stream ~]$ sudo vi /etc/fail2ban/jail.conf
# "bantime" is the number of seconds that a host is banned.
bantime  = 10m

# A host is banned if it has generated "maxretry" during the last "findtime"
# seconds.
findtime  = 1m

# "maxretry" is the number of failures before a host get banned.
maxretry = 3

[audran@stream ~]$ sudo systemctl restart fail2ban
```