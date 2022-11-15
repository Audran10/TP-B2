# Reseau et infra
# I - Dumb switch
## 3) Setup topologique 1

Une fois les VPCS branchés au switch, on définit leur IP static :

- PC1 :
```
ip 10.1.1.1/24
```
On vérifie :
```
PC1> show ip

NAME        : PC1[1]
IP/MASK     : 10.1.1.1/24
```

- PC2 :
```
PC2> ip 10.1.1.2/24
```
On vérifie :
```
PC2> show ip

NAME        : PC2[1]
IP/MASK     : 10.1.1.2/24
```

- On test que les deux VPCS peuvent se ping :
```
PC2> ping 10.1.1.1

84 bytes from 10.1.1.1 icmp_seq=1 ttl=64 time=0.127 ms
84 bytes from 10.1.1.1 icmp_seq=2 ttl=64 time=0.315 ms
84 bytes from 10.1.1.1 icmp_seq=3 ttl=64 time=0.228 ms
84 bytes from 10.1.1.1 icmp_seq=4 ttl=64 time=0.282 ms
```

# II - VLAN
## 3) Setup topologique 2 

### A) Adressage  
<br />
- On garde les configs des VPCS 1 et 2 de la partie précédente. Ensuite
on branche un 3eme VPC sur le switch. De la même manière que dans la partie précédente, on lui donne une ip static => ``ip 10.1.1.3/24``

<br />

- On vérie que les VPCS se ping :

PC3 vers PC1 :
```
PC3> ping 10.1.1.1

84 bytes from 10.1.1.1 icmp_seq=1 ttl=64 time=0.124 ms
84 bytes from 10.1.1.1 icmp_seq=2 ttl=64 time=0.238 ms
84 bytes from 10.1.1.1 icmp_seq=3 ttl=64 time=0.219 ms
```

PC3 vers PC2 :
```
PC3> ping 10.1.1.2

84 bytes from 10.1.1.2 icmp_seq=1 ttl=64 time=0.165 ms
84 bytes from 10.1.1.2 icmp_seq=2 ttl=64 time=0.222 ms
```
Tous les VPCS se ping.

<br />

### B) Configuration des VLANs
<br />

- On déclare les VLANs sur le switch :

```
IOU1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
IOU1(config)#vlan 10
IOU1(config-vlan)#name clients
IOU1(config-vlan)#exit
IOU1(config)#vlan 20
IOU1(config-vlan)#name admins
IOU1(config-vlan)#exit
IOU1(config)#exit
```
Ici on nomme le VLAN 10 clients et le VLAN 20 admins. On vérifie les modifications et on voit quelles ont été appliquées :

```
IOU1#show vlan

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/0, Et0/1, Et0/2, Et0/3
                                                Et1/0, Et1/1, Et1/2, Et1/3
                                                Et2/0, Et2/1, Et2/2, Et2/3
                                                Et3/0, Et3/1, Et3/2, Et3/3
10   clients                          active
20   admins                           active
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup

VLAN Type  SAID       MTU   Parent RingNo BridgeNo Stp  BrdgMode Trans1 Trans2
---- ----- ---------- ----- ------ ------ -------- ---- -------- ------ ------
1    enet  100001     1500  -      -      -        -    -        0      0
10   enet  100010     1500  -      -      -        -    -        0      0
20   enet  100020     1500  -      -      -        -    -        0      0
1002 fddi  101002     1500  -      -      -        -    -        0      0
1003 tr    101003     1500  -      -      -        -    -        0      0
1004 fdnet 101004     1500  -      -      -        ieee -        0      0
1005 trnet 101005     1500  -      -      -        ibm  -        0      0


Primary Secondary Type              Ports
------- --------- ----------------- ------------------------------------------
```

- On ajoute les ports du switch dans les bons VLANs :

VPC 1 (branché sur Ethernet0/1) et VPC 2 (branché sur Ethernet0/2) dans le VLAN10 :
```
IOU1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
IOU1(config)#interface Ethernet0/1
IOU1(config-if)#switchport mode access
IOU1(config-if)#switchport access vlan 10
IOU1(config-if)#exit
IOU1(config)#int Ethernet0/2
IOU1(config-if)#switchport mode access
IOU1(config-if)#switchport access vlan 10
IOU1(config-if)#exit
```

VPC 3 (branché sur Ethernet0/3) dans le VLAN 20 :
```
IOU1(config)#int Ethernet0/3
IOU1(config-if)#switchport mode access
IOU1(config-if)#switchport access vlan 20
IOU1(config-if)#exit
```

Les modifications ont bien été ajoutées :
```
IOU1#show vlan br

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Et0/0, Et1/0, Et1/1, Et1/2
                                                Et1/3, Et2/0, Et2/1, Et2/2
                                                Et2/3, Et3/0, Et3/1, Et3/2
                                                Et3/3
10   clients                          active    Et0/1, Et0/2
20   admins                           active    Et0/3
```
<br />

### C) Vérification
<br />

- Ping entre VPC 1 et VPC 2 :
```
PC1> ping 10.1.1.2

84 bytes from 10.1.1.2 icmp_seq=1 ttl=64 time=0.122 ms
84 bytes from 10.1.1.2 icmp_seq=2 ttl=64 time=0.214 ms
84 bytes from 10.1.1.2 icmp_seq=3 ttl=64 time=0.207 ms
```

- Ping entre VPC 1 et VPC 3 :
```
PC1> ping 10.1.1.3

host (10.1.1.3) not reachable
```

Tout à bien été paramétré.

<br />

# Routing
Tout d'abord, on donne modifie les ip de chaque machine avec la méthode utilisée précédemment :

- VPC1 => ``10.5.10.1/24``
- VPC2 => ``10.5.10.2/24``
- VPC admin => ``10.5.20.1/24``
- Serveur web => ``10.5.30.1/24``

Pour le serveur web :

```
sudo nano /etc/sysconfig/network-scripts/ifcfg-enp0s3
```

```
DEVICE=enp0s3

BOOTPROTO=static
ONBOOT=yes

IPADDR=10.5.30.1
NETMASK=255.255.255.0
```

```
sudo systemctl restart NetworkManager
```

On rajoute maintenant le VLAN 30 dont fera partie le serveur web avec encore une fois la même méthodologie qu'à la partie II-B)

Ensuite, on passe en mode trunk le port qui pointe vers le routeur :

```
# conf t
(config)# interface Ethernet0/0
(config-if)# switchport trunk encapsulation dot1q
(config-if)# switchport mode trunk
(config-if)# switchport trunk allowed vlan add 10,20,30
(config-if)# exit
(config)# exit
```

Enfin, il ne reste plus qu'à créer les sous-interfaces pour le routeur et leur attribuer une ip :

Tout d'abord, il faut activer l'interface fa0/0 :
```
# conf t
R1(config)# int fa0/0
R1(config-subif)# no shut
```

Sous-interface 0.10 pour VLAN 10 :

```
# conf t

(config)# interface fastEthernet 0/0.10
R1(config-subif)# encapsulation dot1Q 10
R1(config-subif)# ip addr 10.5.10.254 255.255.255.0 
R1(config-subif)# exit
```

Sous-interface 0.20 pour VLAN 20 :
```
(config)# interface fastEthernet 0/0.20
R1(config-subif)# encapsulation dot1Q 20
R1(config-subif)# ip addr 10.5.20.254 255.255.255.0 
R1(config-subif)# exit
```

Sous-interface 0.30 pour VLAN 30 :
```
(config)# interface fastEthernet 0/0.30
R1(config-subif)# encapsulation dot1Q 30
R1(config-subif)# ip addr 10.5.30.254 255.255.255.0 
R1(config-subif)# exit
```

On vérifie que les machines peuvent ping le routeur sur l'IP dans leur réseau :
Pour client1 :
```
PC1> ping 10.5.10.254

84 bytes from 10.5.10.254 icmp_seq=1 ttl=255 time=10.209 ms
84 bytes from 10.5.10.254 icmp_seq=2 ttl=255 time=10.868 ms
84 bytes from 10.5.10.254 icmp_seq=3 ttl=255 time=2.966 ms
84 bytes from 10.5.10.254 icmp_seq=4 ttl=255 time=7.090 ms
84 bytes from 10.5.10.254 icmp_seq=5 ttl=255 time=0.881 ms
```

Pour admin1 :
```
admin1> ping 10.5.20.254

10.5.20.254 icmp_seq=1 timeout
84 bytes from 10.5.20.254 icmp_seq=2 ttl=255 time=2.127 ms
84 bytes from 10.5.20.254 icmp_seq=3 ttl=255 time=4.231 ms
84 bytes from 10.5.20.254 icmp_seq=4 ttl=255 time=0.962 ms
84 bytes from 10.5.20.254 icmp_seq=5 ttl=255 time=7.747 ms
```

Pour web_server1 :
```
[audran@localhost ~]$ ping 10.5.30.254
64 bytes from 10.5.30.254 icmp_seq=1 ttl=255 time=10.128 ms
64 bytes from 10.5.30.254 icmp_seq=2 ttl=255 time=5.281 ms
64 bytes from 10.5.30.254 icmp_seq=3 ttl=255 time=2.542 ms
64 bytes from 10.5.30.254 icmp_seq=4 ttl=255 time=8.297 ms
```

Les configs ont été bien réalisée, il ne manque plus que les différentes machines puissent communiquer entre elles. Pour cela, on donne à chaque machine un passerelle par défaut :

Pour client1 et client 2 :

```
#ip 10.5.10.1/24 10.5.10.254
```
```
#ip 10.5.10.2/24 10.5.10.254
```

Pour admin1 :

```
#ip 10.5.20.1/24 10.5.20.254
```

Pour web_server1 :
```
sudo nano /etc/sysconfig/network-scripts/ifcfg-enp0s3
```

```
DEVICE=enp0s3

BOOTPROTO=static
ONBOOT=yes

IPADDR=10.5.30.1
NETMASK=255.255.255.0

GATEWAY=10.5.30.254
```

Toutes les machines peuvent maintenant se ping entre elles :
- client1 vers web_server1 :
```
PC1> ping 10.5.30.1

84 bytes from 10.5.30.1 icmp_seq=1 ttl=63 time=21.103 ms
84 bytes from 10.5.30.1 icmp_seq=2 ttl=63 time=17.435 ms
84 bytes from 10.5.30.1 icmp_seq=3 ttl=63 time=14.405 ms
84 bytes from 10.5.30.1 icmp_seq=4 ttl=63 time=15.673 ms
84 bytes from 10.5.30.1 icmp_seq=5 ttl=63 time=16.675 ms
```

- admin1 vers client1 :
```
admin1> ping 10.5.10.1

84 bytes from 10.5.10.1 icmp_seq=1 ttl=63 time=36.720 ms
84 bytes from 10.5.10.1 icmp_seq=2 ttl=63 time=17.722 ms
84 bytes from 10.5.10.1 icmp_seq=3 ttl=63 time=19.612 ms
84 bytes from 10.5.10.1 icmp_seq=4 ttl=63 time=21.637 ms
84 bytes from 10.5.10.1 icmp_seq=5 ttl=63 time=15.709 ms
```

admin1 vers web_server1 :
```
admin1> ping 10.5.30.1

84 bytes from 10.5.30.1 icmp_seq=1 ttl=63 time=20.599 ms
84 bytes from 10.5.30.1 icmp_seq=2 ttl=63 time=21.566 ms
84 bytes from 10.5.30.1 icmp_seq=3 ttl=63 time=17.815 ms
84 bytes from 10.5.30.1 icmp_seq=4 ttl=63 time=15.608 ms
84 bytes from 10.5.30.1 icmp_seq=5 ttl=63 time=24.715 ms
```

<br />

# IV - NAT
### A) Ajoutez le noeud Cloud à la topo
<br />

Une fois le cloud branché avec le routeur (port eth1 pour le cloud vers fa0/1 pour le routeur), on récupère une IP côté routeur grâce au dhcp :

```
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#int fa0/1
R1(config-if)#ip address dhcp
R1(config-if)#no shut
```

Une adresse IP a bien été attribuée :
```
*Mar  1 02:50:07.803: %DHCP-6-ADDRESS_ASSIGN: Interface FastEthernet0/1 assigned DHCP address 10.0.3.16, mask 255.255.255.0, hostname R1
```

On peut maintenant ping 1.1.1.1 :
```
R1#ping 1.1.1.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 1.1.1.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 16/32/48 ms
```

### B) Configurez le NAT
<br />

On commence par définir nos interfaces inside et outside.

Outside pour fa0/1 :

```
R1(config)#int fa0/1
R1(config-if)#ip nat outside
R1(config-if)#exit
```

Inside pour fa0/0.10, fa0/0.20, fa0/0.30 :
```
R1(config)#int fa0/0.10
R1(config-subif)#ip nat inside
R1(config-subif)#exit
R1(config)#int fa0/0.20
R1(config-subif)#ip nat inside
R1(config-subif)#exit
R1(config)#int fa0/0.30
R1(config-subif)#ip nat inside
R1(config-subif)#exit
```

On définit une liste où le traffic sera autorisé :

```
R1(config)#access-list 1 permit any
```

Puis on configure le NAT :
```
R1(config)#ip nat inside source list 1 interface fa0/1 overload
```

### C) Test
On configure un DNS sur chaque machine et on test un ping sur un nom de domaine :

- client 1 :
```
PC1> ip dns 1.1.1.1

PC1> ping google.com
google.com resolved to 216.58.213.78

google.com icmp_seq=1 timeout
google.com icmp_seq=2 timeout
google.com icmp_seq=3 timeout
google.com icmp_seq=4 timeout
google.com icmp_seq=5 timeout
```

- client2 :
```
PC2> ip dns 1.1.1.1

PC2> ping google.com
google.com resolved to 142.250.75.238

google.com icmp_seq=1 timeout
google.com icmp_seq=2 timeout
google.com icmp_seq=3 timeout
google.com icmp_seq=4 timeout
google.com icmp_seq=5 timeout
```

- admin1 :
```
admin1> ip dns 1.1.1.1

admin1> ping youtube.com
youtube.com resolved to 142.250.75.238

youtube.com icmp_seq=1 timeout
youtube.com icmp_seq=2 timeout
youtube.com icmp_seq=3 timeout
youtube.com icmp_seq=4 timeout
youtube.com icmp_seq=5 timeout
```

- web-server 1 :
```
[audran@localhost ~]$ sudo nano /etc/resolv.conf
```
Et dans le fichier on écrit :
```
namespace 1.1.1.1
```

<br />

# V - Add a building
### A) Show running-config de tous les équipements

Pour le routeur :
```
R1#show running-config

hostname R1
!
interface FastEthernet0/0
 no ip address
 duplex auto
 speed auto
!
interface FastEthernet0/0.10
 encapsulation dot1Q 10
 ip address 10.5.10.254 255.255.255.0
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet0/0.20
 encapsulation dot1Q 20
 ip address 10.5.20.254 255.255.255.0
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet0/0.30
 encapsulation dot1Q 30
 ip address 10.5.30.254 255.255.255.0
 ip nat inside
 ip virtual-reassembly
!
interface FastEthernet0/1
 ip address dhcp
 ip nat outside
 ip virtual-reassembly
 duplex auto
 speed auto
!
ip nat inside source list 1 interface FastEthernet0/1 overload
!
access-list 1 permit any
end
```

Pour le switch 1 :
```
IOU1#show running-config

hostname IOU1

interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet1/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet1/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
end
```

Pour le switch situé en dessous du switch 1 dans la topologie 5 :
```
IOU3#show running-config

hostname IOU3
!
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/1
 switchport access vlan 10
 switchport mode access
!
interface Ethernet0/2
 switchport access vlan 10
 switchport mode access
!
interface Ethernet0/3
 switchport access vlan 20
 switchport mode access
!
interface Ethernet1/0
 switchport access vlan 30
 switchport mode access

end
```

Pour le switch de l'autre bâtiment :

```
IOU2#show running-config

hostname IOU2
!
interface Ethernet0/0
 switchport trunk encapsulation dot1q
 switchport mode trunk
!
interface Ethernet0/1
 switchport access vlan 10
 switchport mode access
!
interface Ethernet0/2
 switchport access vlan 10
 switchport mode access
!
interface Ethernet0/3
 switchport access vlan 10
 switchport mode access
!
interface Ethernet1/0
 switchport access vlan 10
 switchport mode access
!
end
```

<br />

### B) Serveur DHCP dans le nouveau bâtiment

On réutilise un serveur DHCP qu'on a déjà monté et on test si les clients peuvent récupérer une IP en DHCP : 

```
PC4> ip dhcp
DDORA IP 10.5.10.4/24 GW 10.5.10.254

PC4> show ip

NAME        : PC4[1]
IP/MASK     : 10.5.10.4/24
GATEWAY     : 10.5.10.254
DNS         : 1.1.1.1
DHCP SERVER : 10.5.10.253
DHCP LEASE  : 595, 600/300/525
MAC         : 00:50:79:66:68:04
LPORT       : 20034
RHOST:PORT  : 127.0.0.1:20035
MTU         : 1500
```

Le client récupère bien une IP grâce au serveur DHCP 10.5.10.253 qu'on vient de mettre en place.

<br />

### C) Vérifications

- On a vu juste avant que le client récupérait bien une IP en DHCP

- Le VPC 4 peut ping le serveur web :
```
PC4> ping 10.5.30.1

84 bytes from 10.5.30.1 icmp_seq=1 ttl=63 time=19.863 ms
84 bytes from 10.5.30.1 icmp_seq=2 ttl=63 time=17.709 ms
84 bytes from 10.5.30.1 icmp_seq=3 ttl=63 time=18.798 ms
84 bytes from 10.5.30.1 icmp_seq=4 ttl=63 time=11.105 ms
84 bytes from 10.5.30.1 icmp_seq=5 ttl=63 time=18.807 ms
```
- On peut ping 8.8.8.8 :
```
[audran@localhost ~]$ ping 8.8.8.8
64 bytes from 8.8.8.8: icmp_seq=2 ttl=247 time=28.8 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=247 time=25.3 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=247 time=26.9 ms
64 bytes from 8.8.8.8: icmp_seq=5 ttl=247 time=31.4 ms
```

- On peut ping google.com :
```
[audran@localhost ~]$ ping 8.8.8.8
64 bytes from par10s41-in-f14.1e100.net (142.250.75.238): icmp_seq=2 ttl=247 time=30.8 ms
64 bytes from par10s41-in-f14.1e100.net (142.250.75.238): icmp_seq=2 ttl=247 time=27.2 ms
64 bytes from par10s41-in-f14.1e100.net (142.250.75.238): icmp_seq=2 ttl=247 time=29.5 ms
64 bytes from par10s41-in-f14.1e100.net (142.250.75.238): icmp_seq=2 ttl=247 time=31.7 ms
```
