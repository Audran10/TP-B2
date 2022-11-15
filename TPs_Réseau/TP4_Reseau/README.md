# TP4 : TCP, UDP et services réseau

## I - First steps
Youtube, MyCanal, Discord, GitLab, Outlook

1)
- Gitlab => TCP : On se connecte à 172.65.251.78 au port 443. Le port local ouvert est le port 1703

- Discord => TCP : On se connecte à 162.159.130.234 au port 443. Le port local ouvert est le port 1025

- Youtube => UDP : On se connecte à 77.136.192.81 au port 443. Le port local ouvert est le port 61845

- MyCanal => TCP : On se connecte à 185.86.252.41 au port 443. Le port local ouvert est le port 12967

- Outlook => TCP : On se connecte à 52.97.219.194 au port 443. Le port local ouvert est le port 14065
<br><br>
2) La commande à taper dans mon OS (Windows) :
```
netstat -n -b -p tcp
```
(A changer avec UDP pour Youtube)
 - Gitlab : 
```
  TCP    10.33.19.94:12586      172.65.251.78:443      ESTABLISHED
 [chrome.exe]
```
 - discord :
 ```
   Proto  Adresse locale         Adresse distante       État
  TCP    10.33.19.94:1025       162.159.130.234:443    ESTABLISHED
 [Discord.exe]
 ```

 - youtube :
 ```
   UDP    10.33.19.94:12586      77.136.192.81:443      ESTABLISHED
 [chrome.exe]
 ```

 - MyCanal :
 ```
   TCP    10.33.19.94:12586      185.86.252.41:443      ESTABLISHED
 [chrome.exe]
 ```

 - Outlook :
 ```
   TCP    10.33.19.94:14066      52.97.219.194:443      ESTABLISHED
 [OUTLOOK.EXE]
 ```
<br><br>

 ## II. Mise en place
 ## 1) SSH
1)
- SSH utilise TCP. On veut protéger nos données et notre vie privée alors il est logique d'utiliser TCP pour sa fiabilité, sa sécurité.

- On voit que le "SYN SYNACK ACK" se déroule des lignes 1 à 3 dans le fichier pcap. Pour le SYN c'est un échange de notre machine dans lequel elle demande l'autorisation vers 68.232.34.52. Le SYNACK fait le trajet inverse, 68.232.34.52 accepte notre demande et demande la même question à notre machine. Enfin, le ACK n'est autre que notre machine qui accepte la demande. Comme dit à la question précédente, cette échange utilise bien le protocol TCP.

- Le FINACK se situe aux lignes 7 et 8 de notre fichier. A la ligne 7, 68.232.34.52 veut mettre fin au ssh donc à la ligne 8 c'est la même chose mais du côté de notre machine.

2) 
- netstat -a -b : depuis notre machine
```
TCP    10.3.1.3:50194         10.3.1.11:ssh          ESTABLISHED
```

- ss | grep -i ssh : depuis notre VM
```
tcp   ESTAB  0      52                       10.3.1.11:ssh       10.3.1.3:50134
```

## 2) NFS

## III - DNS
- IP du serveur DNS: 8.8.8.8 
- Port : 37281
