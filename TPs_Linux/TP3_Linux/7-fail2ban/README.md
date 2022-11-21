# Module 7 : Fail2Ban

Fail2Ban c'est un peu le cas d'école de l'admin Linux, je vous laisse Google pour le mettre en place.

C'est must-have sur n'importe quel serveur à peu de choses près. En plus d'enrayer les attaques par bruteforce, il limite aussi l'imact sur les performances de ces attaques, en bloquant complètement le trafic venant des IP considérées comme malveillantes

Faites en sorte que :

- si quelqu'un se plante 3 fois de password pour une co SSH en moins de 1 minute, il est ban
```
[audran@db ~]$ sudo vi /etc/fail2ban/jail.local

# "bantime" is the number of seconds that a host is banned.
bantime  = 10m

# A host is banned if it has generated "maxretry" during the last "findtime"
# seconds.
findtime  = 1m

# "maxretry" is the number of failures before a host get banned.
maxretry = 3


[audran@db ~]$ sudo systemctl restart fail2ban
```
- vérifiez que ça fonctionne en vous faisant ban
```
[audran@web ~]$ ssh audran@10.102.1.12
audran@10.102.1.12's password:
audran@10.102.1.12's password:
audran@10.102.1.12's password:
audran@10.102.1.12: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).

[audran@db ~]$ sudo fail2ban-client status sshd
`- Actions
   |- Currently banned: 1
   |- Total banned:     1
   `- Banned IP list:   10.102.1.11
```
- afficher la ligne dans le firewall qui met en place le ban
```
# The simplest action to take: ban only
action_ = %(banaction)s[port="%(port)s", protocol="%(protocol)s", chain="%(chain)s"]
```
- lever le ban avec une commande liée à fail2ban
```
[audran@db ~]$ sudo fail2ban-client unban 10.102.1.11

[audran@web ~]$ ssh audran@10.102.1.12
audran@10.102.1.12's password:
Last failed login: Thu Nov 17 18:02:52 CET 2022 from 10.102.1.11 on ssh:notty
There were 3 failed login attempts since the last successful login.
Last login: Thu Nov 17 18:01:21 2022 from 10.102.1.11
[audran@db ~]$
```

> Vous pouvez vous faire ban en effectuant une connexion SSH depuis `web.tp2.linux` vers `db.tp2.linux` par exemple, comme ça vous gardez intacte la connexion de votre PC vers `db.tp2.linux`, et vous pouvez continuer à bosser en SSH.

