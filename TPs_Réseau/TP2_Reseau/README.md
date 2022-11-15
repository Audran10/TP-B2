# TP 2 - Ethernet, IP, et ARP
(Tp sur powerShell)
## I - Setup
<br>
Q1 : Les IP choisies sont :

- IP1 : 192.168.1.132/26  
- IP2 : 192.168.1.133/26

L'adresse de réseau est : 192.168.1.128/26

L'adresse de broadcast est : 192.168.1.191/26
<br><br>
Pour changer l'IP en ligne de commande :

New-NetIPAddress -InterfaceIndex “4” -IPAddress 192.168.1.132 -PrefixLength 26

Q2 : ping 192.168.1.133 -> Le ping fonctionne

Q3 : Pour le ping c'est un ICMP de type "Echo Request" et pour le pong c'est un ICMP de type "Echo Reply"
<br><br>

## II - ARP my bro
Q1 : 
- arp -a (Pour afficher la table ARP)
- 192.168.1.133         10-62-e5-e2-90-3d (MAC de mon binôme)
- Passerelle par défaut. . . . . . . . . : 10.33.19.254 (ipconfig /all pour l'obtenir)

Q2 : 
```
Interface : 192.168.1.132 --- 0x4
  Adresse Internet      Adresse physique      Type
  192.168.1.133         10-62-e5-e2-90-3d     dynamique
  192.168.1.191         ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique

Interface : 192.168.160.1 --- 0xd
  Adresse Internet      Adresse physique      Type
  192.168.160.254       00-50-56-e6-ad-fe     dynamique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique

Interface : 192.168.171.1 --- 0xf
  Adresse Internet      Adresse physique      Type
  192.168.171.254       00-50-56-ea-c5-4e     dynamique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique

Interface : 10.33.19.94 --- 0x10
  Adresse Internet      Adresse physique      Type
  10.33.19.254          00-c0-e7-e0-04-4e     dynamique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
```
- arp -d (Pour vider la table ARP)
- arp -a (Pour l'afficher une nouvelle fois)

Il ne reste plus que ça :

```
Interface : 192.168.160.1 --- 0xd
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 192.168.171.1 --- 0xf
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique

Interface : 10.33.19.94 --- 0x10
  Adresse Internet      Adresse physique      Type
  10.33.19.254          00-c0-e7-e0-04-4e     dynamique
  224.0.0.22            01-00-5e-00-00-16     statique
```
On constate qu'il n'y a plus aucune trace de nôtre binôme
- ping 192.168.1.133 -> Retour de la table 
<br><br>

Q3 : L'adresse source est 192.168.1.132 (la mienne) et l'adresse destination est 192.168.1.133 (celle de mon binôme)
<br><br>

## III-DHCP you too my brooo
Q1 : Les 4 trames DHCP lors d'un échange DHCP sont :
1) Discover
      - adresse source 0.0.0.0 / adresse distination 255.255.255.255
2) Offer
      - adresse source 10.33.19.255 / adresse destination 10.33.18.144
3) Request
      - adresse source 0.0.0.0 / adresse distination 255.255.255.255
4) ACK
      - adresse source 10.33.19.255 / adresse destination 10.33.18.144
<br><br>

Q2 : 
- 1 -> 10.33.18.144 (Ip à utiliser)
- 2 -> 10.33.19.254 (Ip de la passerelle)
- 3 -> 10.33.19.254 (Ip d'un serveur DNS)

<br><br>

## IV. Avant-goût TCP et UDP
  - Ip de youtube : 173.194.183.234
  - Port : 443 (port https)
