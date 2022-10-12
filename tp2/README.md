

# I. Setup IP

Le lab, il vous faut deux machines : 

- les deux machines doivent être connectées physiquement
- vous devez choisir vous-mêmes les IPs à attribuer sur les interfaces réseau, les contraintes :
  - IPs privées (évidemment n_n)
  - dans un réseau qui peut contenir au moins 1000 adresses IP (il faut donc choisir un masque adapté)
  - oui c'est random, on s'exerce c'est tout, p'tit jog en se levant c:
  - le masque choisi doit être le plus grand possible (le plus proche de 32 possible) afin que le réseau soit le plus petit possible

🌞 **Mettez en place une configuration réseau fonctionnelle entre les deux machines**

- vous renseignerez dans le compte rendu :
  - les deux IPs choisies, en précisant le masque
  - l'adresse de réseau
  - l'adresse de broadcast

  ```
  192.168.128.10/22 et 192.168.128.240/22
  192.168.128.0 adresse du réseau
  192.168.131.255 adresse du broadcast
  255.255.252.0 le masque car 2^(32-10)=1024, Donc il y a 1024 combinaison possible donc 1022 adresse IP adressable sur ce reseau 
  ```
- vous renseignerez aussi les commandes utilisées pour définir les adresses
 IP *via* la ligne de commande
 ```
$ netsh int ip set address "Ethernet" address=192.168.128.240 mask=255.255.252.0 gateway=192.168.128.10
$ netsh int ipv4 set dnsservers name="Ethernet"  source=static address="8.8.8.8" validate=no
```
> Rappel : tout doit être fait *via* la ligne de commandes. Faites-vous du bien, et utilisez Powershell plutôt que l'antique cmd sous Windows svp.

🌞 **Prouvez que la connexion est fonctionnelle entre les deux machines**

- un `ping` suffit !
```
PS C:\Users\erwan> ping 192.168.128.10

Envoi d’une requête 'Ping'  192.168.128.10 avec 32 octets de données :
Réponse de 192.168.128.10 : octets=32 temps=1 ms TTL=128
Réponse de 192.168.128.10 : octets=32 temps=2 ms TTL=128
Réponse de 192.168.128.10 : octets=32 temps=1 ms TTL=128
Réponse de 192.168.128.10 : octets=32 temps=2 ms TTL=128

Statistiques Ping pour 192.168.128.10:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 1ms, Maximum = 2ms, Moyenne = 1ms
```

🌞 **Wireshark it**

- `ping` ça envoie des paquets de type ICMP (c'est pas de l'IP, c'est un de ses frères)
  - les paquets ICMP sont encapsulés dans des trames Ethernet, comme les paquets IP
  - il existe plusieurs types de paquets ICMP, qui servent à faire des trucs différents
- **déterminez, grâce à Wireshark, quel type de paquet ICMP est envoyé par `ping`**
  - pour le ping que vous envoyez
  - et le pong que vous recevez en retour
  ```txt
  les packets envoyés par 'ping' son tu type echo request pour le ping et echo reply pour le pong.

> Vous trouverez sur [la page Wikipedia de ICMP](https://en.wikipedia.org/wiki/Internet_Control_Message_Protocol) un tableau qui répertorie tous les types ICMP et leur utilité

🦈 **PCAP qui contient les paquets ICMP qui vous ont permis d'identifier les types ICMP**

# II. ARP my bro

ARP permet, pour rappel, de résoudre la situation suivante :

- pour communiquer avec quelqu'un dans un LAN, il **FAUT** connaître son adresse MAC
- on admet un PC1 et un PC2 dans le même LAN :
  - PC1 veut joindre PC2
  - PC1 et PC2 ont une IP correctement définie
  - PC1 a besoin de connaître la MAC de PC2 pour lui envoyer des messages
  - **dans cette situation, PC1 va utilise le protocole ARP pour connaître la MAC de PC2**
  - une fois que PC1 connaît la mac de PC2, il l'enregistre dans sa **table ARP**

🌞 **Check the ARP table**

- utilisez une commande pour afficher votre table ARP
- déterminez la MAC de votre binome depuis votre table ARP
- déterminez la MAC de la *gateway* de votre réseau
  - celle de votre réseau physique, WiFi, genre YNOV, car il n'y en a pas dans votre ptit LAN
  - c'est juste pour vous faire manipuler un peu encore :)
  ```
  $ arp -a
  Interface : 192.168.128.10 --- 0x13
  Adresse Internet      Adresse physique      Type
  192.168.128.10       84-69-93-52-2e-d1     dynamique
  224.0.0.22            01-00-5e-00-00-16     statique

> Il peut être utile de ré-effectuer des `ping` avant d'afficher la table ARP. En effet : les infos stockées dans la table ARP ne sont stockées que temporairement. Ce laps de temps est de l'ordre de ~60 secondes sur la plupart de nos machines.

🌞 **Manipuler la table ARP**

- utilisez une commande pour vider votre table ARP
- prouvez que ça fonctionne en l'affichant et en constatant les changements
- ré-effectuez des pings, et constatez la ré-apparition des données dans la table ARP
```
$ arp -d
$ arp -a

Interface : 192.168.128.10 --- 0x13
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique

$ ping 192.168.128.10

Envoi d’une requête 'Ping'  192.168.128.10 avec 32 octets de données :
Réponse de 192.168.128.10 : octets=32 temps=1 ms TTL=128
Réponse de 192.168.128.10 : octets=32 temps=2 ms TTL=128
Réponse de 192.168.128.10 : octets=32 temps=1 ms TTL=128
Réponse de 192.168.128.10 : octets=32 temps=2 ms TTL=128

Statistiques Ping pour 192.168.128.10:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 1ms, Maximum = 2ms, Moyenne = 1ms

$ arp -a

Interface : 192.168.128.10 --- 0x13
  Adresse Internet      Adresse physique      Type
  192.168.128.10       84-69-93-52-2e-d1     dynamique
  224.0.0.22            01-00-5e-00-00-16     statique
```
> Les échanges ARP sont effectuées automatiquement par votre machine lorsqu'elle essaie de joindre une machine sur le même LAN qu'elle. Si la MAC du destinataire n'est pas déjà dans la table ARP, alors un échange ARP sera déclenché.

🌞 **Wireshark it**

- vous savez maintenant comment forcer un échange ARP : il sufit de vider la table ARP et tenter de contacter quelqu'un, l'échange ARP se fait automatiquement
- mettez en évidence les deux trames ARP échangées lorsque vous essayez de contacter quelqu'un pour la "première" fois
  - déterminez, pour les deux trames, les adresses source et destination
  - déterminez à quoi correspond chacune de ces adresses
  
``` 
source: 08:8f:c3:46:88:ec  est ma machine qui questionne au serveur DHCP si l'adresse ip qu'il a est b1 
destination: HP_52:2e:d1 puis il est repond que oui c'est bon
```
🦈 **PCAP qui contient les trames ARP**

> L'échange ARP est constitué de deux trames : un ARP broadcast et un ARP reply.

# III. DHCP you too my brooo


*DHCP* pour *Dynamic Host Configuration Protocol* est notre p'tit pote qui nous file des IPs quand on arrive dans un réseau, parce que c'est chiant de le faire à la main :)

Quand on arrive dans un réseau, notre PC contacte un serveur DHCP, et récupère généralement 3 infos :

- **1.** une IP à utiliser
- **2.** l'adresse IP de la passerelle du réseau
- **3.** l'adresse d'un serveur DNS joignable depuis ce réseau

L'échange DHCP  entre un client et le serveur DHCP consiste en 4 trames : **DORA**, que je vous laisse chercher sur le web vous-mêmes : D

🌞 **Wireshark it**

- identifiez les 4 trames DHCP lors d'un échange DHCP
  - mettez en évidence les adresses source et destination de chaque trame
- identifiez dans ces 4 trames les informations **1**, **2** et **3** dont on a parlé juste au dessus
 ```
 DORA car Discorver Offer Request Ack  
 discover le serveur decouvre ce pc
 offer lui donne une ip 
 request lui donne l'adresse de la paserelle
 ack lui donne un serveur dns joignable dupuis ce reseau
 ```

🦈 **PCAP qui contient l'échange DORA**

> **Soucis** : l'échange DHCP ne se produit qu'à la première connexion. **Pour forcer un échange DHCP**, ça dépend de votre OS. Sur **GNU/Linux**, avec `dhclient` ça se fait bien. Sur **Windows**, le plus simple reste de définir une IP statique pourrie sur la carte réseau, se déconnecter du réseau, remettre en DHCP, se reconnecter au réseau. Sur **MacOS**, je connais peu mais Internet dit qu'c'est po si compliqué, appelez moi si besoin.
