# TP 3 : On va router des trucs
## I - ARP
## 1) Echange ARP
1) Ping de John vers Marcel :
```
[audran@localhost ~]$ ping 10.3.1.12
PING 10.3.1.12 (10.3.1.12) 56(84) bytes of data.
64 bytes from 10.3.1.12: icmp_seq=1 ttl=64 time=0.411 ms
64 bytes from 10.3.1.12: icmp_seq=2 ttl=64 time=0.353 ms
64 bytes from 10.3.1.12: icmp_seq=3 ttl=64 time=0.487 ms
^C
--- 10.3.1.12 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2048ms
```

2) Table ARP de John :
```
[audran@localhost ~]$ ip neigh show
10.3.1.12 dev enp0s3 lladdr 08:00:27:b8:2a:59 STALE
10.3.1.3 dev enp0s3 lladdr 0a:00:27:00:00:05 REACHA
```
Table ARP de Marcel :
```
[audran@localhost ~]$ ip neigh show
10.3.1.11 dev enp0s8 lladdr 08:00:27:86:55:16 STALE
10.3.1.5 dev enp0s8 lladdr 0a:00:27:00:00:0e REACHABLE
```
On voit qu'il y a bien eu un échange entre les deux machines.

Pour voir la MAC de Marcel dans la table ARP de John :
```
[audran@localhost network-scripts]$ ip neigh show 10.3.1.12
10.3.1.12 dev enp0s3 lladdr 08:00:27:b8:2a:59 STALE
```

Pour voir la MAC de Marcel depuis la table ARP de Marcel => ip a 
```
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:b8:2a:59 brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.12/24 brd 10.3.1.255 scope global noprefixroute enp0s3
```

L'adresse MAC de Marcel est identique, l'info est donc correcte.
<br><br>

## 2) Analyse de trames
1) Pour faire la capture de trame :
```
sudo tcpdump -i enp0s8 -c 10 -w tp3_arp.pcapng
```
2) Pour vider les tables ARP :
```
sudo ip neigh flush all
```
Puis lancement d'un ping entre les deux machines.
<br><br>

## II - Routage
## 1) Mise en place du routage
1) 
- Activer le routage sur le noeud router :
```
$ sudo firewall-cmd --add-masquerade --zone=public
$ sudo firewall-cmd --add-masquerade --zone=public --permanent
```
-  Rajout d'un adapter dans VBox pour le router
- Changement d'ip des 2 interfaces du router :
```
sudo nano /etc/sysconfig/network-scripts/ifcfg-enp0s3
sudo nano /etc/sysconfig/network-scripts/ifcfg-enp0s8
```
- Ajout des 2 routes statique chez John et Marcel:
```
sudo nano /etc/sysconfig/network-scripts/route-enp0s3
10.3.1.11/24 via 10.3.1.254 dev enp0s3
```
```
sudo nano /etc/sysconfig/network-scripts/route-enp0s3
10.3.2.12/24 via 10.3.2.254 dev enp0s3
```

On fait un test de ping pour vérifier que John puisse joindre Marcel :
```
[audran@localhost ~]$ ping 10.3.2.12
PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=2.24 ms
64 bytes from 10.3.2.12: icmp_seq=2 ttl=63 time=1.28 ms
64 bytes from 10.3.2.12: icmp_seq=3 ttl=63 time=1.44 ms
64 bytes from 10.3.2.12: icmp_seq=4 ttl=63 time=1.37 ms
64 bytes from 10.3.2.12: icmp_seq=5 ttl=63 time=1.47 ms
```
Et on vérifie aussi que Marcel puisse joindre John :
```
[audran@localhost ~]$ ping 10.3.1.11
PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.
64 bytes from 10.3.1.11: icmp_seq=1 ttl=63 time=1.26 ms
64 bytes from 10.3.1.11: icmp_seq=2 ttl=63 time=2.71 ms
64 bytes from 10.3.1.11: icmp_seq=3 ttl=63 time=3.31 ms
64 bytes from 10.3.1.11: icmp_seq=4 ttl=63 time=2.73 ms
64 bytes from 10.3.1.11: icmp_seq=5 ttl=63 time=2.58 ms
```

## 2) Analyse de trames

| ordre | type  trame    | IP  source | MAC  source              | IP destination | MAC  destination         |
|-------|----------------|------------|--------------------------|----------------|--------------------------|
| 1     | Requête ARP    | 10.3.1.11  | John 08:00:27:b8:2a:59  | 10.3.2.254     | Broadcast FF:FF:FF:FF    |
| 2     | Réponse  ARP   | 10.3.2.12  | Marcel 08:00:27:b8:2a:59 | 10.3.1.11      | John 08:00:27:b8:2a:59   |
| 3     | Ping (request) | 10.3.2.254 | Router 08:00:27:7a:7d:ee | 10.3.2.12      | Marcel 08:00:27:b8:2a:59 |
| 4     | Pong (reply)   | 10.3.2.12  | Marcel 08:00:27:b8:2a:59 | 10.3.2.254     | Router 08:00:27:7a:7d:ee |
| 5     | Requête  ARP   | 10.3.2.12  | Marcel 08:00:27:b8:2a:59 | 10.3.2.254     | Router 08:00:27:7a:7d:ee |
| 6     | Réponse  ARP   | 10.3.2.254 | Router 08:00:27:7a:7d:ee | 10.3.2.12      | Marcel 08:00:27:b8:2a:59 |

## 3) Accès internet
1) Donnez un accès internet à vos machines

On ajoute une carte NAT sur le router via VBox. On test la connexion internet :

```
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=119 time=13.3 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=119 time=12.1 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=119 time=13.8 ms
```

On rajoute une route pour John et Marcel :

John : sudo nano /etc/sysconfig/network
```
# Created by anaconda
GATEWAY=10.3.1.254
```

Marcel : sudo nano /etc/sysconfig/network
```
# Created by anaconda
GATEWAY=10.3.2.254
```

On effectue la vérification : 

John : ip r s 
```
[audran@localhost ~]$ sudo ip r s
[sudo] password for audran:
default via 10.3.1.254 dev enp0s3 proto static metric 100
10.3.1.0/24 dev enp0s3 proto kernel scope link src 10.3.1.11 metric 100
10.3.2.0/24 via 10.3.1.254 dev enp0s3 proto static metric 100
```

Marcel : ip r s
```
[audran@localhost ~]$ sudo ip r s
[sudo] password for audran:
default via 10.3.2.254 dev enp0s3 proto static metric 100
10.3.1.11 via 10.3.2.254 dev enp0s3 proto static metric 100
10.3.2.0/24 dev enp0s3 proto kernel scope link src 10.3.2.12 metric 100
```

On test avec la machine de John un ping vers une adresse => ping 8.8.8.8
```
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=118 time=21.2 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=118 time=17.2 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=118 time=17.7 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=118 time=17.7 ms
```

On modifie le fichier le fichier resolv.conf => sudo nano /etc/resolv.conf
```
# Generated by NetworkManager
nameserver 8.8.8.8
nameserver 8.8.4.4
nameserver 1.1.1.1
```
Vérification de la résolution de noms => dig google.com
```
; <<>> DiG 9.16.23-RH <<>> google.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 46389
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;google.com.                    IN      A

;; ANSWER SECTION:
google.com.             236     IN      A       142.250.179.110

;; Query time: 29 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Sun Oct 09 14:55:55 CEST 2022
;; MSG SIZE  rcvd: 55
```

Ping d'un nom de domaine => ping google.com
```
PING google.com (142.250.75.238) 56(84) bytes of data.
64 bytes from par10s41-in-f14.1e100.net (142.250.75.238): icmp_seq=1 ttl=118 time=14.9 ms
64 bytes from par10s41-in-f14.1e100.net (142.250.75.238): icmp_seq=2 ttl=118 time=17.0 ms
64 bytes from par10s41-in-f14.1e100.net (142.250.75.238): icmp_seq=3 ttl=118 time=19.1 ms
64 bytes from par10s41-in-f14.1e100.net (142.250.75.238): icmp_seq=4 ttl=118 time=16.4 ms
```
Cette manipulation est effective chez John mais aussi Marcel.

2) Analyse de trames

On ping 8.8.8.8 depuis John => sudo tcpdump -i enp0s3 -w tp3_routage_internet.pcapng

On nous donne la même MAC pour les 2 IP :

| ordre | type  trame    | IP  source | MAC  source              | IP destination | MAC  destination         |
|-------|----------------|------------|--------------------------|----------------|--------------------------|
| 1     | Ping    | 10.3.1.11  | John 08:00:27:b8:2a:59  | 8.8.8.8     | 08:00:27:b8:2a:59  |
| 2     | Pong   | 8.8.8.8  | 08:00:27:b8:2a:59 | 10.3.1.11      | John 08:00:27:b8:2a:59   |

## III - DHCP
## 1) 1. Mise en place du serveur DHCP
1) Installation du serveur DHCP sur John :
```
- sudo dnf install dhcp-server
- sudo nano /etc/dhcp/dhcpd.conf
```
Fichier dhcpd.conf :
```
default-lease-time 900;
max-lease-time 10800;
ddns-update-style none;
authoritative;
subnet 10.3.1.0 netmask 255.255.255.0 {
  range 10.3.1.12 10.3.1.253;
  option routers 10.3.1.254; 
  option subnet-mask 255.255.255.0;
  option domain-name-servers 8.8.8.8;
}
```
Puis on tape ces commandes :
```
- sudo systemctl enable --now dhcpd
- sudo systemctl status dhcpd
```
```
Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; enabled; vendor preset: disabled)
Active: active (running) since Sun 2022-10-09 21:19:44 CEST; 17min ago
```
Définir une IP dynamique pour Bob :
```
sudo nano /etc/sysconfig/network-scripts/ifcfg-enp0s3
```
Dans ce fichier :
```
NAME=enp0s8
DEVICE=enp0s8

BOOTPROTO=dhcp
ONBOOT=yes
```
On tape :
```
sudo systemctl restart NetworkManager
```
Puis => ip a :
```
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
link/ether 08:00:27:b8:2a:59 brd ff:ff:ff:ff:ff:ff
inet 10.3.2.12/24 brd 10.3.2.255 scope global noprefixroute enp0s3
```

2) Améliorer la configuration du DHCP :

Dans le fichier dhcpd.conf :
```
default-lease-time 900;
max-lease-time 10800;
ddns-update-style none;
authoritative;
subnet 10.3.1.0 netmask 255.255.255.0 {
  range 10.3.1.12 10.3.1.253;
  option routers 10.3.1.254; 
  option subnet-mask 255.255.255.0;
  option domain-name-servers 8.8.8.8;
}
```

On récupère une IP pour Bob :
```
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
link/ether 08:00:27:b8:2a:59 brd ff:ff:ff:ff:ff:ff
inet 10.3.2.12/24 brd 10.3.2.255 scope global noprefixroute enp0s3
```
Ping de Bob vers sa passerelle :
```
PING 10.3.1.254 (10.3.1.254) 56(84) bytes of data.
64 bytes from 10.3.1.254: icmp_seq=1 ttl=64 time=0.294 ms
64 bytes from 10.3.1.254: icmp_seq=1 ttl=63 time=0.412 ms
```

Route définie par défaut de Bob => ip r s
```
[audran@localhost ~]$ ip r s
default via 10.3.1.254 dev enp0s3 proto dhcp src 10.3.1.12 metric 100
```

On ecrit la commande :
```
dig facebook.com
```
Et on ping l'IP que l'on obtient :
```
PING 157.240.21.35 (157.240.21.35) 56(84) bytes of data.
64 bytes from 157.240.21.35: icmp_seq=1 ttl=56 time=273 ms
64 bytes from 157.240.21.35: icmp_seq=1 ttl=55 time=274 ms (DUP!)
64 bytes from 157.240.21.35: icmp_seq=1 ttl=55 time=274 ms (DUP!)
```

La commande dig => dig twitter.com
```

; <<>> DiG 9.16.23-RH <<>> twitter.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 7706
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;twitter.com.                   IN      A

;; ANSWER SECTION:
twitter.com.            196     IN      A       104.244.42.193
twitter.com.            196     IN      A       104.244.42.1

;; Query time: 43 msec
;; SERVER: 192.168.1.254#53(192.168.1.254)
;; WHEN: Sun Oct 09 23:19:48 CEST 2022
;; MSG SIZE  rcvd: 72
```

Ping de Bob vers un nom de domaine :
```
PING google.com (216.58.198.206) 56(84) bytes of data.
64 bytes from par10s27-in-f14.1e100.net (216.58.198.206): icmp_seq=1 ttl=118 time=26.0 ms
64 bytes from par10s27-in-f206.1e100.net (216.58.198.206): icmp_seq=1 ttl=117 time=26.4 ms
```

## 2) Analyse de trames
=> sudo tcpdump -i enp0s8 -w tp3_dhcp.pcapng

Pour demander une nouvelle Ip :
   - on supprime l'ancienne avec => sudo dhclient -r
   - On en redemande une nouvelle avec => sudo dhclient
