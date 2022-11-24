# TP2 : Gestion de service

Dans ce TP on va s'orienter sur l'**utilisation des systèmes GNU/Linux** comme un outil pour **faire tourner des services**. C'est le principal travail de l'administrateur système : fournir des services.

Ces services, on fait toujours la même chose avec :

- **installation** (opération ponctuelle)
- **configuration** (opération ponctuelle)
- **maintien en condition opérationnelle** (opération continue, tant que le service est actif)
- **renforcement niveau sécurité** (opération ponctuelle et continue : on conf robuste et on se tient à jour)

**Dans cette première partie, on va voir la partie installation et configuration.** Peu importe l'outil visé, de la base de données au serveur cache, en passant par le serveur web, le serveur mail, le serveur DNS, ou le serveur privé de ton meilleur jeu en ligne, c'est toujours pareil : install into conf.

On abordera la sécurité et le maintien en condition opérationelle dans une deuxième partie.

**On va apprendre à maîtriser un peu ces étapes, et pas simplement suivre la doc.**

On va maîtriser le service fourni :

- manipulation du service avec systemd
- quelle IP et quel port il utilise
- quels utilisateurs du système sont mobilisés
- quels processus sont actifs sur la machine pour que le service soit actif
- gestion des fichiers qui concernent le service et des permissions associées
- gestion avancée de la configuration du service

---

Bon le service qu'on va setup c'est NextCloud. **JE SAIS** ça fait redite avec l'an dernier, me tapez pas. ME TAPEZ PAS.  

Mais vous inquiétez pas, on va pousser le truc, on va faire évoluer l'install, l'architecture de la solution. Cette première partie de TP, on réalise une install basique, simple, simple, basique, la version *vanilla* un peu. Ce que vous êtes censés commencer à maîtriser (un peu, faites moi plais).

Refaire une install guidée, ça permet de s'exercer à faire ça proprement dans un cadre, bien comprendre, et ça me fait un pont pour les B1C aussi :)

On va faire évoluer la solution dans la suite de ce TP.

# Sommaire

- [TP2 : Gestion de service](#tp2--gestion-de-service)
- [Sommaire](#sommaire)
- [0. Prérequis](#0-prérequis)
  - [Checklist](#checklist)
- [I. Un premier serveur web](#i-un-premier-serveur-web)
  - [1. Installation](#1-installation)
  - [2. Avancer vers la maîtrise du service](#2-avancer-vers-la-maîtrise-du-service)
- [II. Une stack web plus avancée](#ii-une-stack-web-plus-avancée)
  - [1. Intro blabla](#1-intro-blabla)
  - [2. Setup](#2-setup)
    - [A. Base de données](#a-base-de-données)
    - [B. Serveur Web et NextCloud](#b-serveur-web-et-nextcloud)
    - [C. Finaliser l'installation de NextCloud](#c-finaliser-linstallation-de-nextcloud)

# 0. Prérequis

➜ Machines Rocky Linux

➜ Un unique host-only côté VBox, ça suffira. **L'adresse du réseau host-only sera `10.102.1.0/24`.**

➜ Chaque **création de machines** sera indiquée par **l'emoji 🖥️ suivi du nom de la machine**

➜ Si je veux **un fichier dans le rendu**, il y aura l'**emoji 📁 avec le nom du fichier voulu**. Le fichier devra être livré tel quel dans le dépôt git, ou dans le corps du rendu Markdown si c'est lisible et correctement formaté.

## Checklist

A chaque machine déployée, vous **DEVREZ** vérifier la 📝**checklist**📝 :

- [x] IP locale, statique ou dynamique
- [x] hostname défini
- [x] firewall actif, qui ne laisse passer que le strict nécessaire
- [x] SSH fonctionnel avec un échange de clé
- [x] accès Internet (une route par défaut, une carte NAT c'est très bien)
- [x] résolution de nom
- [x] SELinux désactivé (vérifiez avec `sestatus`, voir [mémo install VM tout en bas](https://gitlab.com/it4lik/b2-reseau-2022/-/blob/main/cours/memo/install_vm.md#4-pr%C3%A9parer-la-vm-au-clonage))

**Les éléments de la 📝checklist📝 sont STRICTEMENT OBLIGATOIRES à réaliser mais ne doivent PAS figurer dans le rendu.**

![Checklist](./pics/checklist_is_here.jpg)

# I. Un premier serveur web

## 1. Installation

🖥️ **VM web.tp2.linux**

| Machine         | IP            | Service     |
|-----------------|---------------|-------------|
| `web.tp2.linux` | `10.102.1.11` | Serveur Web |

🌞 **Installer le serveur Apache**

- paquet `httpd`

```
[audran@web ~]$ sudo dnf install httpd
Complete!
```
- la conf se trouve dans `/etc/httpd/`
  - le fichier de conf principal est `/etc/httpd/conf/httpd.conf`
  - je vous conseille **vivement** de virer tous les commentaire du fichier, à défaut de les lire, vous y verrez plus clair
    - avec `vim` vous pouvez tout virer avec `:g/^ *#.*/d`
    ```
    [audran@web ~]$ sudo vim /etc/httpd/conf/httpd.conf
    `:g/^ *#.*/d`
    ```

> Ce que j'entends au-dessus par "fichier de conf principal" c'est que c'est **LE SEUL** fichier de conf lu par Apache quand il démarre. C'est souvent comme ça : un service ne lit qu'un unique fichier de conf pour démarrer. Cherchez pas, on va toujours au plus simple. Un seul fichier, c'est simple.  
**En revanche** ce serait le bordel si on mettait toute la conf dans un seul fichier pour pas mal de services.  
Donc, le principe, c'est que ce "fichier de conf principal" définit généralement deux choses. D'une part la conf globale. D'autre part, il inclut d'autres fichiers de confs plus spécifiques.  
On a le meilleur des deux mondes : simplicité (un seul fichier lu au démarrage) et la propreté (éclater la conf dans plusieurs fichiers).

🌞 **Démarrer le service Apache**

- le service s'appelle `httpd` (raccourci pour `httpd.service` en réalité)
  - démarrez le
  ```
  [audran@web ~]$ sudo systemctl start httpd
  [audran@web ~]$ sudo systemctl is-active httpd
  active
  ```
  - faites en sorte qu'Apache démarre automatique au démarrage de la machine
  ```
  [audran@web ~]$ sudo systemctl enable httpd
  Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.

  [audran@web ~]$ sudo systemctl is-enabled httpd
  enabled
  ```
  - ouvrez le port firewall nécessaire
    - utiliser une commande `ss` pour savoir sur quel port tourne actuellement Apache
    - [une petite portion du mémo est consacrée à `ss`](https://gitlab.com/it4lik/b2-linux-2021/-/blob/main/cours/memo/commandes.md#r%C3%A9seau)
    ```
    [audran@web ~]$ sudo ss -alnpt
    LISTEN       0        511           *:80             *:*  
    users:(("httpd",pid=35103,fd=4),("httpd",pid=35102,fd=4),("httpd",pid=35101,fd=4),("httpd",pid=35099,fd=4))

    [audran@web ~]$ sudo firewall-cmd --add-port=80/tcp --permanent
    success
    [audran@web ~]$ sudo firewall-cmd --reload
    success
    [audran@web ~]$ sudo firewall-cmd --list-all
    ports: 80/tcp
    ```

**En cas de problème** (IN CASE OF FIIIIRE) vous pouvez check les logs d'Apache :

```bash
# Demander à systemd les logs relatifs au service httpd
$ sudo journalctl -xe -u httpd

# Consulter le fichier de logs d'erreur d'Apache
$ sudo cat /var/log/httpd/error_log

# Il existe aussi un fichier de log qui enregistre toutes les requêtes effectuées sur votre serveur
$ sudo cat /var/log/httpd/access_log
```

🌞 **TEST**

- vérifier que le service est démarré
```
[audran@web ~]$ sudo systemctl is-active httpd
active
```
- vérifier qu'il est configuré pour démarrer automatiquement
```
[audran@web ~]$ sudo systemctl is-enabled httpd
enabled
```
- vérifier avec une commande `curl localhost` que vous joignez votre serveur web localement
```
[audran@web ~]$ curl 10.102.1.11:80
<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
    <meta name='viewport' content='width=device-width, initial-scale=1'>
    <title>HTTP Server Test Page powered by: Rocky Linux</title>
    <style type="text/css">
```

- vérifier avec votre navigateur (sur votre PC) que vous accéder à votre serveur web
```
PS C:\Users\Audran> curl 10.102.1.11:80
curl : HTTP Server Test Page
This page is used to test the proper operation of an HTTP server after it has been installed on a Rocky Linux system.
If you can read this page, it means that the software it working correctly.
```

## 2. Avancer vers la maîtrise du service

🌞 **Le service Apache...**

- affichez le contenu du fichier `httpd.service` qui contient la définition du service Apache
```
[audran@web ~]$ systemctl cat httpd
[Unit]
Description=The Apache HTTP Server
Wants=httpd-init.service
After=network.target remote-fs.target nss-lookup.target httpd-init.service
Documentation=man:httpd.service(8)

[Service]
Type=notify
Environment=LANG=C

ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND
ExecReload=/usr/sbin/httpd $OPTIONS -k graceful
# Send SIGWINCH for graceful stop
KillSignal=SIGWINCH
KillMode=mixed
PrivateTmp=true
OOMPolicy=continue

[Install]
WantedBy=multi-user.target
```

🌞 **Déterminer sous quel utilisateur tourne le processus Apache**

- mettez en évidence la ligne dans le fichier de conf principal d'Apache (`httpd.conf`) qui définit quel user est utilisé
```
[audran@web ~]$ cat /etc/httpd/conf/httpd.conf
User apache
```
- utilisez la commande `ps -ef` pour visualiser les processus en cours d'exécution et confirmer que apache tourne bien sous l'utilisateur mentionné dans le fichier de conf
```
root       35099       1  0 10:16 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache     35100   35099  0 10:16 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache     35101   35099  0 10:16 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache     35102   35099  0 10:16 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
apache     35103   35099  0 10:16 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
```
Le User est bien apache.

- la page d'accueil d'Apache se trouve dans `/usr/share/testpage/`
  - vérifiez avec un `ls -al` que tout son contenu est **accessible en lecture** à l'utilisateur mentionné dans le fichier de conf

```
[audran@web ~]$ ls -al /usr/share/testpage/
-rw-r--r--.  1 root root 7620 Jul  6 04:37 index.html
```

🌞 **Changer l'utilisateur utilisé par Apache**

- créez un nouvel utilisateur
  - pour les options de création, inspirez-vous de l'utilisateur Apache existant
    - le fichier `/etc/passwd` contient les informations relatives aux utilisateurs existants sur la machine
    - servez-vous en pour voir la config actuelle de l'utilisateur Apache par défaut
    ```
    [audran@web ~]$ sudo useradd audran10 -m -d /usr/share/httpd -s /sbin/nologin
    [audran@web ~]$ cat /etc/passwd
    apache:x:48:48:Apache:/usr/share/httpd:/sbin/nologin
    audran10:x:1001:1001::/usr/share/httpd:/sbin/nologin
    ```
- modifiez la configuration d'Apache pour qu'il utilise ce nouvel utilisateur
```
[audran@web ~]$ sudo vim /etc/httpd/conf/httpd.conf

User audran10
Group audran10
```
- redémarrez Apache
```
[audran@web ~]$ sudo systemctl restart httpd
```
- utilisez une commande `ps` pour vérifier que le changement a pris effet
```
root       35522       1  0 11:26 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
audran10   35523   35522  0 11:26 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
audran10   35524   35522  0 11:26 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
audran10   35525   35522  0 11:26 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
audran10   35526   35522  0 11:26 ?        00:00:00 /usr/sbin/httpd -DFOREGROUND
```

🌞 **Faites en sorte que Apache tourne sur un autre port**

- modifiez la configuration d'Apache pour lui demander d'écouter sur un autre port de votre choix
```
[audran@web ~]$ sudo vim /etc/httpd/conf/httpd.conf
Listen 100
```
- ouvrez ce nouveau port dans le firewall, et fermez l'ancien
```
[audran@web ~]$ sudo firewall-cmd --remove-port=80/tcp --permanent
[audran@web ~]$ sudo firewall-cmd --add-port=100/tcp --permanent
[audran@web ~]$ sudo firewall-cmd --reload
[audran@web ~]$ sudo firewall-cmd --list-all
ports: 100/tcp
```
- redémarrez Apache
```
[audran@web ~]$ sudo systemctl restart httpd
```
- prouvez avec une commande `ss` que Apache tourne bien sur le nouveau port choisi
```
[audran@web ~]$ sudo systemctl restart httpd
[audran@web ~]$ sudo ss -alpnt
LISTEN          0               511                 *:100                      *:*
 users:(("httpd",pid=35763,fd=4),("httpd",pid=35762,fd=4),("httpd",pid=35761,fd=4),("httpd",pid=35757,fd=4))
```
- vérifiez avec `curl` en local que vous pouvez joindre Apache sur le nouveau port
```
[audran@web ~]$ curl 10.102.1.11:100
<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
```
- vérifiez avec votre navigateur que vous pouvez joindre le serveur sur le nouveau port
```
PS C:\Users\Audran> curl 10.102.1.11:100
curl : HTTP Server Test Page
This page is used to test the proper operation of an HTTP server after it has been installed on a Rocky Linux system. If you can read this page, it means that the software it working correctly.
```

📁 **Fichier `/etc/httpd/conf/httpd.conf`**

# II. Une stack web plus avancée

⚠⚠⚠ **Réinitialiser votre conf Apache avant de continuer** ⚠⚠⚠  
En particulier :

- reprendre le port par défaut
- reprendre l'utilisateur par défaut

## 1. Intro blabla

**Le serveur web `web.tp2.linux` sera le serveur qui accueillera les clients.** C'est sur son IP que les clients devront aller pour visiter le site web.  

**Le service de base de données `db.tp2.linux` sera uniquement accessible depuis `web.tp2.linux`.** Les clients ne pourront pas y accéder. Le serveur de base de données stocke les infos nécessaires au serveur web, pour le bon fonctionnement du site web.

---

Bon le but de ce TP est juste de s'exercer à faire tourner des services, un serveur + sa base de données, c'est un peu le cas d'école. J'ai pas envie d'aller deep dans la conf de l'un ou de l'autre avec vous pour le moment, on va se contenter d'une conf minimale.

Je vais pas vous demander de coder une application, et cette fois on se contentera pas d'un simple `index.html` tout moche et on va se mettre dans la peau de l'admin qui se retrouve avec une application à faire tourner. **On va faire tourner un [NextCloud](https://nextcloud.com/).**

En plus c'est utile comme truc : c'est un p'tit serveur pour héberger ses fichiers via une WebUI, style Google Drive. Mais on l'héberge nous-mêmes :)

---

Le flow va être le suivant :

➜ **on prépare d'abord la base de données**, avant de setup NextCloud

- comme ça il aura plus qu'à s'y connecter
- ce sera sur une nouvelle machine `db.tp2.linux`
- il faudra installer le service de base de données, puis lancer le service
- on pourra alors créer, au sein du service de base de données, le nécessaire pour NextCloud

➜ **ensuite on met en place NextCloud**

- on réutilise la machine précédente avec Apache déjà installé, ce sera toujours Apache qui accueillera les requêtes des clients
- mais plutôt que de retourner une bête page HTML, NextCloud traitera la requête
- NextCloud, c'est codé en PHP, il faudra donc **installer une version de PHP précise** sur la machine
- on va donc : install PHP, configurer Apache, récupérer un `.zip` de NextCloud, et l'extraire au bon endroit !

![NextCloud install](./pics/nc_install.png)

## 2. Setup

🖥️ **VM db.tp2.linux**

**N'oubliez pas de dérouler la [📝**checklist**📝](#checklist).**

| Machines        | IP            | Service                 |
|-----------------|---------------|-------------------------|
| `web.tp2.linux` | `10.102.1.11` | Serveur Web             |
| `db.tp2.linux`  | `10.102.1.12` | Serveur Base de Données |

### A. Base de données

🌞 **Install de MariaDB sur `db.tp2.linux`**

- déroulez [la doc d'install de Rocky](https://docs.rockylinux.org/guides/database/database_mariadb-server/)
- je veux dans le rendu **toutes** les commandes réalisées
```
[audran@db ~]$ sudo dnf install mariadb-server
Complete!

[audran@db ~]$ sudo systemctl enable mariadb
Created symlink /etc/systemd/system/mysql.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/mysqld.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/multi-user.target.wants/mariadb.service → /usr/lib/systemd/system/mariadb.service.

[audran@db ~]$ sudo systemctl start mariadb
[audran@db ~]$ mysql_secure_installation
```
- vous repérerez le port utilisé par MariaDB avec une commande `ss` exécutée sur `db.tp2.linux`
```
[audran@db ~]$ sudo ss -alnpt
LISTEN     0          80                         *:3306                    *:*        users:(("mariadbd",pid=12807,fd=15))
```
  - il sera nécessaire de l'ouvrir dans le firewall
  ```
  [audran@db ~]$ sudo firewall-cmd --add-port=3306/tcp --permanent
  [audran@db ~]$ sudo firewall-cmd --reload
  [audran@db ~]$ sudo firewall-cmd --list-all
  ports: 3306/tcp
  ```

> La doc vous fait exécuter la commande `mysql_secure_installation` c'est un bon réflexe pour renforcer la base qui a une configuration un peu *chillax* à l'install.

🌞 **Préparation de la base pour NextCloud**

- une fois en place, il va falloir préparer une base de données pour NextCloud :
  - connectez-vous à la base de données à l'aide de la commande `sudo mysql -u root -p`
  - exécutez les commandes SQL suivantes :

```sql

CREATE USER 'nextcloud'@'10.102.1.11' IDENTIFIED BY 'azerty';

CREATE DATABASE IF NOT EXISTS nextcloud CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;

GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'10.102.1.11';

FLUSH PRIVILEGES;
```

> Par défaut, vous avez le droit de vous connecter localement à la base si vous êtes `root`. C'est pour ça que `sudo mysql -u root` fonctionne, sans nous demander de mot de passe. Evidemment, n'importe quelles autres conditions ne permettent pas une connexion aussi facile à la base.

🌞 **Exploration de la base de données**

- afin de tester le bon fonctionnement de la base de données, vous allez essayer de vous connecter, comme NextCloud le fera :
  - depuis la machine `web.tp2.linux` vers l'IP de `db.tp2.linux`
  - utilisez la commande `mysql` pour vous connecter à une base de données depuis la ligne de commande
    - par exemple `mysql -u <USER> -h <IP_DATABASE> -p`
    ```
    [audran@web ~]$ mysql -u nextcloud -h 10.102.1.12 -p
    Enter password:
    Welcome to the MariaDB monitor.
    ```
    - si vous ne l'avez pas, installez-là
    - vous pouvez déterminer dans quel paquet est disponible la commande `mysql` en saisissant `dnf provides mysql`
    ```
    [audran@web ~]$ dnf provides mysql
    mysql-8.0.30-3.el9_0.x86_64 
    ```
- **donc vous devez effectuer une commande `mysql` sur `web.tp2.linux`**
- une fois connecté à la base, utilisez les commandes SQL fournies ci-dessous pour explorer la base

```sql
SHOW DATABASES;
USE <DATABASE_NAME>;
SHOW TABLES;
```

🌞 **Trouver une commande SQL qui permet de lister tous les utilisateurs de la base de données**

> Les utilisateurs de la base de données sont différents des utilisateurs du système Rocky Linux qui porte la base. Les utilisateurs de la base définissent des identifiants utilisés pour se connecter à la base afin d'y voir ou d'y modifier des données.

Une fois qu'on s'est assurés qu'on peut se co au service de base de données depuis `web.tp2.linux`, on peut continuer.

### B. Serveur Web et NextCloud

⚠️⚠️⚠️ **N'OUBLIEZ PAS de réinitialiser votre conf Apache avant de continuer. En particulier, remettez le port et le user par défaut.**

🌞 **Install de PHP**

```bash
[audran@web ~]$ sudo dnf config-manager --set-enabled crb
[audran@web ~]$ sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm -y
Complete!

[audran@web ~]$ dnf module list php
Extra Packages for Enterprise Linux 9 - x86_64                              397 kB/s |  11 MB     00:29
Remi s Modular repository for Enterprise Linux 9 - x86_64                   1.1 kB/s | 833  B     00:00

[audran@web ~]$ sudo dnf module enable php:remi-8.1 -y
Complete!

[audran@web ~]$ sudo dnf install -y php81-php
Complete!
```

🌞 **Install de tous les modules PHP nécessaires pour NextCloud**

```bash
[audran@web ~]$ sudo dnf install -y libxml2 openssl php81-php php81-php-ctype php81-php-curl php81-php-gd php81-php-iconv php81-php-json php81-php-libxml php81-php-mbstring php81-php-openssl php81-php-posix php81-php-session php81-php-xml php81-php-zip php81-php-zlib php81-php-pdo php81-php-mysqlnd php81-php-intl php81-php-bcmath php81-php-gmp
Complete!
```

🌞 **Récupérer NextCloud**

- créez le dossier `/var/www/tp2_nextcloud/`
```
[audran@web ~]$ sudo mkdir /var/www/tp2_nextcloud
```
  - ce sera notre *racine web* (ou *webroot*)
  - l'endroit où le site est stocké quoi, on y trouvera un `index.html` et un tas d'autres marde, tout ce qui constitue NextClo :D
- récupérer le fichier suivant avec une commande `curl` ou `wget` : https://download.nextcloud.com/server/prereleases/nextcloud-25.0.0rc3.zip
```
[audran@web ~]$ curl -SLO https://download.nextcloud.com/server/prereleases/nextcloud-25.0.0rc3.zip
100  168M  100  168M    0     0   345k      0  0:08:18  0:08:18 --:--:--  401k
```
- extrayez tout son contenu dans le dossier `/var/www/tp2_nextcloud/` en utilisant la commande `unzip`
  - installez la commande `unzip` si nécessaire
  ```
  [audran@web ~]$ sudo dnf install unzip
  Complete!
  ```
  - vous pouvez extraire puis déplacer ensuite, vous prenez pas la tête
  ```
  [audran@web ~]$ sudo unzip nextcloud-25.0.0rc3.zip -d /var/www/tp2_nextcloud/
  ```
  - contrôlez que le fichier `/var/www/tp2_nextcloud/index.html` existe pour vérifier que tout est en place
  ```
  [audran@web ~]$ cd /var/www/tp2_nextcloud/nextcloud/
  [audran@web nextcloud]$ ls
  index.html
  ```
- assurez-vous que le dossier `/var/www/tp2_nextcloud/` et tout son contenu appartient à l'utilisateur qui exécute le service Apache
```
[audran@web tp2_nextcloud]$ sudo chown -R apache:apache /var/www/tp2_nextcloud/ 
[audran@web tp2_nextcloud]$ ls -al /var/www/tp2_nextcloud/
drwxr-xr-x. 15 apache apache  4096 Nov 17 09:23 .
drwxr-xr-x.  5 root   root      54 Nov 17 01:28 ..
drwxr-xr-x. 47 apache apache  4096 Oct  6 14:47 3rdparty
-rw-r--r--.  1 apache apache   156 Oct  6 14:42 index.html
```
Maintenant, c'est bien le même utilisateur.

> A chaque fois que vous faites ce genre de trucs, assurez-vous que c'est bien ok. Par exemple, vérifiez avec un `ls -al` que tout appartient bien à l'utilisateur qui exécute Apache.

🌞 **Adapter la configuration d'Apache**

- regardez la dernière ligne du fichier de conf d'Apache pour constater qu'il existe une ligne qui inclut d'autres fichiers de conf
- créez en conséquence un fichier de configuration qui porte un nom clair et qui contient la configuration suivante :

```
[audran@web ~]$ sudo nano /etc/httpd/conf/httpd2.conf

<VirtualHost *:80>
  DocumentRoot /var/www/tp2_nextcloud/ # on indique le chemin de notre webroot
  ServerName  web.tp2.linux # on précise le nom que saisissent les clients pour accéder au service
  <Directory /var/www/tp2_nextcloud/> # on définit des règles d'accès sur notre webroot
    Require all granted
    AllowOverride All
    Options FollowSymLinks MultiViews
    <IfModule mod_dav.c>
      Dav off
    </IfModule>
  </Directory>
</VirtualHost>
```

🌞 **Redémarrer le service Apache** pour qu'il prenne en compte le nouveau fichier de conf
```
[audran@web ~]$ sudo systemctl restart httpd
```

### C. Finaliser l'installation de NextCloud

➜ **Sur votre PC**

- modifiez votre fichier `hosts` (oui, celui de votre PC, de votre hôte)
  - pour pouvoir joindre l'IP de la VM en utilisant le nom `web.tp2.linux`
  ```
  Dans C:\Windows\System32\drivers\etc\hosts :

  # For example:
  #
  #      102.54.94.97     rhino.acme.com          # source server
  #       38.25.63.10     x.acme.com              # x client host
  10.102.1.11	    web.tp2.linux
  ```
- avec un navigateur, visitez NextCloud à l'URL `http://web.tp2.linux`
  - c'est possible grâce à la modification de votre fichier `hosts`
- on va vous demander un utilisateur et un mot de passe pour créer un compte admin
  - ne saisissez rien pour le moment
- cliquez sur "Storage & Database" juste en dessous
  - choisissez "MySQL/MariaDB"
  - saisissez les informations pour que NextCloud puisse se connecter avec votre base
- saisissez l'identifiant et le mot de passe admin que vous voulez, et validez l'installation

🌴 **C'est chez vous ici**, baladez vous un peu sur l'interface de NextCloud, faites le tour du propriétaire :)

🌞 **Exploration de la base de données**

- connectez vous en ligne de commande à la base de données après l'installation terminée
```
[audran@web ~]$ mysql -u nextcloud -h 10.102.1.12 -p

MariaDB [(none)]> USE nextcloud;
MariaDB [nextcloud]> SHOW TABLES;
+-----------------------------+
| Tables_in_nextcloud         |
+-----------------------------+
| oc_accounts                 |
| oc_accounts_data            |
124 rows in set (0.001 sec)
```
- déterminer combien de tables ont été crées par NextCloud lors de la finalisation de l'installation

  - ***bonus points*** si la réponse à cette question est automatiquement donnée par une requête SQL
