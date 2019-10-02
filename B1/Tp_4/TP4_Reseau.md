# I Mise en place du lab

## 2 Création des VMs

On ping le routeur avec la manip : ping serveur.routeur.
````
[root@serveur simon]# ping serveur.routeur
PING serveur.routeur (10.2.0.254) 56(84) bytes of data.
64 bytes from serveur.routeur (10.2.0.254): icmp_seq=1 ttl=64 time=0.220 ms
64 bytes from serveur.routeur (10.2.0.254): icmp_seq=2 ttl=64 time=0.310 ms
64 bytes from serveur.routeur (10.2.0.254): icmp_seq=3 ttl=64 time=0.311 ms
^C
--- serveur.routeur ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2000ms
rtt min/avg/max/mdev = 0.220/0.280/0.311/0.044 ms
[root@serveur simon]#
````
## 3 Mise en place du routage statique

Pour que client1 ait une route vers net1 et net2, il faut rentrer la commande:
````
nano /etc/sysconfig/network-scripts/route-enp0s3
````
Ensuite on rentrera à l'intérieur : 
````
10.2.0.254/24 via 10.1.0.254 dev enp0s3
````
Puis on effectue un redémarrage avec la commande:
````
systemctl restart network
````
Grâce à cela le client1 a une route pour accéder vers le net2.

Nous n'avons pas besoin de créer une route vers net1 car elle existe dèjà.

Pour server1 on effectue la même manip, à l'exception que l'on rentrera dans le scripts:
````
10.1.0.254/24 via 10.2.0.254 dev enp0s3
````
Ping de client1 vers server1:
````
[root@client simon]# ping 10.2.0.10
PING 10.2.0.10 (10.2.0.10) 56(84) bytes of data.
64 bytes from 10.2.0.10: icmp_seq=1 ttl=63 time=0.641 ms
64 bytes from 10.2.0.10: icmp_seq=2 ttl=63 time=0.492 ms
^C
--- 10.2.0.10 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.492/0.566/0.641/0.078 ms
[root@client simon]#
````
Ping de server1 vers client1:
````
[root@serveur simon]# ping 10.1.0.10
PING 10.1.0.10 (10.1.0.10) 56(84) bytes of data.
64 bytes from 10.1.0.10: icmp_seq=1 ttl=63 time=0.643 ms
64 bytes from 10.1.0.10: icmp_seq=2 ttl=63 time=0.677 ms
^C
--- 10.1.0.10 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1000ms
rtt min/avg/max/mdev = 0.643/0.660/0.677/0.017 ms
[root@serveur simon]#
````
# II Spéléologie réseau

## 1.ARP

### A Manip 1

Pour afficher la table ARP du client1, on rentre la commande :
````
ip neigh show
````
La ligne qu'on peut voir:
````
10.1.0.1 dev enp0s3 lladdr 0a:00:27:00:00:19 REACHABLE
````
C'est la carte réseau: le net1.

On effectue la même chose pour server1, on peut observer 
````
10.2.0.1 dev enp0s3 lladdr 0a:00:27:00:00:0c REACHABLE
````
C'est la carte réseau: le net2

Lorsque client1 ping le server1 on peut voir l'ip du routeur apparaître.
````
10.1.0.254 dev enp0s3 lladdr 08:00:27:25:b1:07 REACHABLE
````
Ca veut dire que lorsqu'on ping le server1 on passe par le routeur qui nous a renvoyé sa MAC et son IP 

La table d'ARP du server1 on voit apparaitre:
````
10.2.0.254 dev enp0s3 lladdr 08:00:27:70:3f:1d REACHABLE
````

C'est la MAC du routeur1 sur le réseau 2. Car pour renvoyer un ping au client1 il faut passer par le routeur1 qu'on doit connaître.

### B Manip 2

Lorsqu'on fait:

````
ip neigh show
````

on voit apparaitre: 

````
10.1.0.1 dev enp0s3 lladdr 0a:00:27:00:00:19 DELAY
````

C'est notre PC hote toutes les cartes réseau y sont reliés car ce sont des VMs.

Apres que le client1 ping server1 sur la table ARP du routeur1 on voit:

````
10.1.0.10 dev enp0s3 lladdr 08:00:27:88:ee:22 REACHABLE
10.2.0.10 dev enp0s8 lladdr 08:00:27:b0:3c:0d REACHABLE
````

On voit l'ip du server1 et client1 ainsi que la MAC des deux. cela s'explique avec le ping du client1 car le router1 doit savoir à quelle adresse doit envoyer le ping et à qu'il doit envoyer la réponse. Pour ce faire il reçoit la MAC et l'IP du client1 et server1.

### C Manip 3

Affichage de la table ARP avec la commande arp -a après vidage:
````
Interface : 10.2.0.1 --- 0xc
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique
  230.0.0.1             01-00-5e-00-00-01     statique

Interface : 169.254.126.1 --- 0x13
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique
  230.0.0.1             01-00-5e-00-00-01     statique

Interface : 10.33.3.217 --- 0x17
  Adresse Internet      Adresse physique      Type
  10.33.3.253           00-12-00-40-4c-bf     dynamique
  224.0.0.22            01-00-5e-00-00-16     statique
  230.0.0.1             01-00-5e-00-00-01     statique
  255.255.255.255       ff-ff-ff-ff-ff-ff     statique

Interface : 10.1.0.1 --- 0x19
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique
  230.0.0.1             01-00-5e-00-00-01     statique
  255.255.255.255       ff-ff-ff-ff-ff-ff     statique
````

Apres quelques minutes on refait arp -a:
````
Interface : 10.2.0.1 --- 0xc
  Adresse Internet      Adresse physique      Type
  10.2.0.255            ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  230.0.0.1             01-00-5e-00-00-01     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique

Interface : 169.254.126.1 --- 0x13
  Adresse Internet      Adresse physique      Type
  169.254.255.255       ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  230.0.0.1             01-00-5e-00-00-01     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
  255.255.255.255       ff-ff-ff-ff-ff-ff     statique

Interface : 10.33.3.217 --- 0x17
  Adresse Internet      Adresse physique      Type
  10.33.0.36            60-67-20-ec-0f-98     dynamique
  10.33.3.253           00-12-00-40-4c-bf     dynamique
  10.33.3.254           94-0c-6d-84-50-c8     dynamique
  10.33.3.255           ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  230.0.0.1             01-00-5e-00-00-01     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
  255.255.255.255       ff-ff-ff-ff-ff-ff     statique

Interface : 10.1.0.1 --- 0x19
  Adresse Internet      Adresse physique      Type
  10.1.0.255            ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  230.0.0.1             01-00-5e-00-00-01     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
  255.255.255.255       ff-ff-ff-ff-ff-ff     statique
````

Les nouveaux IPs qu'on voit apparaître sont toutes les requêtes qui nous sont demandés.

### D Manip 4

on voit apparaître une nouvelle ligne.
````
10.0.4.2 dev enp0s9 lladdr 52:54:00:12:35:02 STALE
10.1.0.1 dev enp0s3 lladdr 0a:00:27:00:00:19 DELAY
````

La machine qui porte l'IP 10.0.4.2 est client1 et on peut dire même dire que c'est sa passerelle.

## 2 Wireshark



