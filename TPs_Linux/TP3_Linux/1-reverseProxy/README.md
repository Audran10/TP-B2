# Module 1 : Reverse Proxy

Un reverse proxy est donc une machine que l'on place devant un autre service afin d'accueillir les clients et servir d'intermédiaire entre le client et le service.

L'utilisation d'un reverse proxy peut apporter de nombreux bénéfices :

- décharger le service HTTP de devoir effectuer le chiffrement HTTPS (coûteux en performances)
- répartir la charge entre plusieurs services
- effectuer de la mise en cache
- fournir un rempart solide entre un hacker potentiel et le service et les données importantes
- servir de point d'entrée unique pour accéder à plusieurs services web

![Not sure](../pics/reverse_proxy.png)

## Sommaire

- [Module 1 : Reverse Proxy](#module-1--reverse-proxy)
  - [Sommaire](#sommaire)
- [I. Intro](#i-intro)
- [II. Setup](#ii-setup)
- [III. HTTPS](#iii-https)

# I. Intro

# II. Setup

🖥️ **VM `proxy.tp3.linux`**

**N'oubliez pas de dérouler la [📝**checklist**📝](#checklist).**

➜ **On utilisera NGINX comme reverse proxy**

- installer le paquet `nginx`

```
[audran@proxy ~]$ sudo dnf install nginx
Complete!
```

- démarrer le service `nginx`

```
[audran@proxy ~]$ sudo systemctl start nginx
```

- utiliser la commande `ss` pour repérer le port sur lequel NGINX écoute

```
[audran@proxy ~]$ sudo ss -alnpt
LISTEN 0      511           0.0.0.0:80          0.0.0.0:*     users:(("nginx",pid=10750,fd=6),("nginx",pid=10749,fd=6))

LISTEN 0      511              [::]:80             [::]:*     users:(("nginx",pid=10750,fd=7),("nginx",pid=10749,fd=7))
```

- ouvrir un port dans le firewall pour autoriser le trafic vers NGINX

```
[audran@proxy ~]$ sudo firewall-cmd --add-port=80/tcp --permanent
[audran@proxy ~]$ sudo firewall-cmd --reload
[audran@proxy ~]$ sudo firewall-cmd --list-all
ports: 80/tcp
```
- utiliser une commande `ps -ef` pour déterminer sous quel utilisateur tourne NGINX

```
[audran@proxy ~]$ ps -ef
root       10749       1  0 12:27 ?        00:00:00 nginx: master process /usr/sbin/nginx
nginx      10750   10749  0 12:27 ?        00:00:00 nginx: worker process
```

- vérifier que le page d'accueil NGINX est disponible en faisant une requête HTTP sur le port 80 de la machine

```
[audran@proxy ~]$ curl 10.102.1.13:80
<!doctype html>
<html>
  <head>
    <meta charset='utf-8'>
```

➜ **Configurer NGINX**

- nous ce qu'on veut, c'pas une page d'accueil moche, c'est que NGINX agisse comme un reverse proxy entre les clients et notre serveur Web
- deux choses à faire :
  - créer un fichier de configuration NGINX
    - la conf est dans `/etc/nginx`
    - procédez comme pour Apache : repérez les fichiers inclus par le fichier de conf principal, et créez votre fichier de conf en conséquence
    ```
    [audran@proxy ~]$ sudo nano /etc/nginx/conf.d/nginx2.conf

    server {
        # On indique le nom que client va saisir pour accéder au service
        # Pas d'erreur ici, c'est bien le nom de web, et pas de proxy qu'on veut ici !
        server_name web.tp2.linux;

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
            proxy_pass http://10.102.1.11:80;
        }

        # Deux sections location recommandés par la doc NextCloud
        location /.well-known/carddav {
        return 301 $scheme://$host/remote.php/dav;
        }

        location /.well-known/caldav {
        return 301 $scheme://$host/remote.php/dav;
        }
    }
    ```
  - NextCloud est un peu exigeant, et il demande à être informé si on le met derrière un reverse proxy
    - y'a donc un fichier de conf NextCloud à modifier
    - c'est un fichier appelé `config.php`
    ```
    [audran@web ~]$ sudo nano /var/www/tp2_nextcloud/config/config.php
      'trusted_domains' =>
    array (
        0 => 'web.tp2.linux',
        1 => '10.102.1.13',
    ),
    ```

Référez-vous à monsieur Google pour tout ça :)

➜ **Modifier votre fichier `hosts` de VOTRE PC**

- pour que le service soit joignable avec le nom `web.tp2.linux`
```
10.102.1.13	web.tp2.linux
```
- c'est à dire que `web.tp2.linux` doit pointer vers l'IP de `proxy.tp3.linux`
- autrement dit, pour votre PC :
  - `web.tp2.linux` pointe vers l'IP du reverse proxy
  - `proxy.tp3.linux` ne pointe vers rien
  - taper `http://web.tp2.linux` permet d'accéder au site (en passant de façon transparente par l'IP du proxy)

> Oui vous ne rêvez pas : le nom d'une machine donnée pointe vers l'IP d'une autre ! Ici, on fait juste en sorte qu'un certain nom permette d'accéder au service, sans se soucier de qui porte réellement ce nom.

✨ **Bonus** : rendre le serveur `web.tp2.linux` injoignable sauf depuis l'IP du reverse proxy. En effet, les clients ne doivent pas joindre en direct le serveur web : notre reverse proxy est là pour servir de serveur frontal.

- faire un `ping` manuel vers l'IP de `proxy.tp3.linux` fonctionne
- faire un `ping` manuel vers l'IP de `web.tp2.linux` ne fonctionne pas

# III. HTTPS

Le but de cette section est de permettre une connexion chiffrée lorsqu'un client se connecte. Avoir le ptit HTTPS :)

Le principe :

- on génère une paire de clés sur le serveur `proxy.tp3.linux`
```
[audran@proxy ~]$ openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout domain.key -out domain.crt
[audran@proxy ~]$ sudo mv domain.crt /etc/pki/tls/certs/
[audran@proxy ~]$ sudo mv domain.key /etc/pki/tls/private/
```
  - une des deux clés sera la clé privée : elle restera sur le serveur et ne bougera jamais
  - l'autre est la clé publique : elle sera stockée dans un fichier appelé *certificat*
    - le *certificat* est donné à chaque client qui se connecte au site
- on ajuste la conf NGINX
  - on lui indique le chemin vers le certificat et la clé privée afin qu'il puisse les utiliser pour chiffrer le trafic
  - on lui demande d'écouter sur le port convetionnel pour HTTPS : 443 en TCP
```
[audran@proxy ~]$ sudo nano /etc/nginx/conf.d/nginx2.conf

server {
# On indique le nom que client va saisir pour accéder au service
# Pas d'erreur ici, c'est bien le nom de web, et pas de proxy qu'on veut ici !
server_name web.tp2.linux;

# Port d'écoute de NGINX
listen 443 ssl http2;
listen [::]:443 ssl http2;

ssl_certificate     /etc/pki/tls/certs/domain.crt;
ssl_certificate_key /etc/pki/tls/private/domain.key;

[audran@proxy ~]$ sudo firewall-cmd --add-port=443/tcp --permanent
[audran@proxy ~]$ sudo firewall-cmd --reload
```

On rajoute cette ligne pour rediriger automatiquement en HTTPS :
```
[audran@proxy ~]$ sudo vim /etc/nginx/conf.d/nginx2.conf
server {
    listen 80 default_server;
    server_name _;
    return 301 https://$host$request_uri;
}
```
Je vous laisse Google vous-mêmes "nginx reverse proxy nextcloud" ou ce genre de chose :)
