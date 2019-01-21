
# I Création et utilisation simples d'une VM CentOS

## 4. Configuration réseau d'une machine CentOS

### a

J'ai utilisé "curl -SL www.google.com"

cela nous affiche le code source de la page google.

### b 

J'ai utilisé le ping de la VM au PC hôte.

[poulet@localhost ~]$ ping 192.168.127.1
PING 192.168.127.1 (192.168.127.1) 56(84) bytes of data.
64 bytes from 192.168.127.1: icmp_seq=1 ttl=128 time=0.163 ms
64 bytes from 192.168.127.1: icmp_seq=2 ttl=128 time=0.208 ms
64 bytes from 192.168.127.1: icmp_seq=3 ttl=128 time=0.223 ms
64 bytes from 192.168.127.1: icmp_seq=4 ttl=128 time=0.275 ms
64 bytes from 192.168.127.1: icmp_seq=5 ttl=128 time=0.262 ms
--- 192.168.127.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4001ms

Puis le ping du PC hôte à la VM:

PS C:\Users\ShaTaNuS> ping 192.168.127.10

Envoi d’une requête 'Ping'  192.168.127.10 avec 32 octets de données :
Réponse de 192.168.127.10 : octets=32 temps<1ms TTL=64
Réponse de 192.168.127.10 : octets=32 temps<1ms TTL=64
Réponse de 192.168.127.10 : octets=32 temps<1ms TTL=64
Réponse de 192.168.127.10 : octets=32 temps<1ms TTL=64

Statistiques Ping pour 192.168.127.10:
Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%)


### c 

On a rentré : ip route

On obtient 3 lignes:

-La premiere "default via 10.0.2.2 dev enp0s3 proto dhcp metric 100".
    C'est l'adresse du pc hôte qui nous permet d'aller sur internet avec la VM. Elle servira de passerelle.

-La deuxieme "10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100".
    C'est la nat local qu'on a créer.

-La troisieme "192.168.127.0/24 dev enp0s8 proto kernel scope link src 192.168.127.10 metric 101".
    C'est le réseau local qu'on a créer nous permettant de communiquer entre le pC hôte et la  VM.

# II Notion de ports et SSH

## 1 Exploration des ports locaux

Lorsque que l'on écrit dans l'invité de commandes :" ss -n "

On obtient : tcp    ESTAB      0      7708             192.168.127.10:22                            192.168.127.1:1236

## 2 SSH

Pour ce connecter en SSH à la VM on écrit: "ssh poulet@192.168.127.10"

Puis on obtient cela : "The authenticity of host '192.168.127.10 (192.168.127.10)' can't be established.
ECDSA key fingerprint is SHA256:XU8+U+pKPEcrHDPrZAQUIoCsLi+lFwyQJr7rqkgYwKE.
Are you sure you want to continue connecting (yes/no)?"

Ou l'on rentre yes et on entre ensuite le password de l'utilisateur.

## 3 Firewall

### A

On rentre dans l'invité nano /etc/ssh/sshd_config

On peut changer le port en port 2222
On enregistre et on remédarre

avec systemctl restart sshd

[root@localhost poulet]# ss -4tlnp
State       Recv-Q Send-Q                       Local Address:Port                                      Peer Address:Port
LISTEN      0      128                                      *:2222                                                 *:*                   users:(("sshd",pid=3480,fd=3))
LISTEN      0      100                              127.0.0.1:25                                                   *:*                   users:(("master",pid=3387,fd=13))

On voit maintenant que l'on écoute sur le port 2222.

Pour ce connecter on va maintenant écrire ssh poulet@192.168.127.10 -p 2222
Sauf que la connection ne marche pas car l'on doit autoriser avec le firewall.

Pour ce faire on va noter firewall-cmd --add-port=2222/tcp --permanent

Puis redémarrer firewall-cmd --reload

[root@localhost poulet]# firewall-cmd --add-port=2222/tcp --permanent
Warning: ALREADY_ENABLED: 2222:tcp
success
[root@localhost poulet]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: ssh dhcpv6-client
  ports: 2222/tcp 2500/tcp 5454/tcp
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:

[root@localhost poulet]# firewall-cmd --reloa
success
[root@localhost poulet]# firewall-cmd --reload
success

### B Netcat


On va noter [root@localhost poulet]# nc -l 5454 pour écouter sur le port 5454

ensuite on va l'activer sur le firewall [root@localhost poulet]# firewall-cmd --add-port=5454/tcp --permanent

et on va ouvrir un autre cmd et rentrer :

nc64 192.168.127.10 5454

Pour voir le pont qui ce créer:

[root@localhost poulet]# ss -4tlnp
State       Recv-Q Send-Q                       Local Address:Port                                      Peer Address:Port
LISTEN      0      128                                      *:2222                                                 *:*                   users:(("sshd",pid=3480,fd=3))
LISTEN      0      100                              127.0.0.1:25                                                   *:*                   users:(("master",pid=3387,fd=13))

# III. Routage statique

##1 Préparation des hôtes

Après avoir désactivé SeLinux comme vu ici nous pouvons commencer cette troisième partie. Pour ceci il nous faut modifier toutes les addresse ip de nos machines pour d'éviter les conflits ou les messages d'erreurs. Voici un petit tableau récapitulatif des modifications à apporter et histoire de ne pas s'emmêler ainsi que désactiver les carte enp0s3 sur nos vm pour utiliser exclusivement le port internet.

[root@localhost poulet]# ifdown enp0s3
Device 'enp0s3' successfully disconnected.

Network 1                   //      Network 2             //     network 12
Windows Pc 1	192.168.101.1	// Windows Pc 2	192.168.102.2	// Windows Pc 1	192.168.112.1

Vm CentOs 1	192.168.101.10	// Vm CentOs 2	192.168.102.10	// Windows Pc 2	192.168.112.2

Une fois le tout fait nous devons nous assurer que tout fonctionne, nous allons donc ping des deux côtés la vm au pc hôte et les deux pc hôte ensemble cela donne donc :

Nous faisons un Ping de Windows 2 à Windows 1

 C:\Users\ShaTaNuS>ping 162.168.112.1
    Envoi d’une requête 'Ping'  162.168.112.1 avec 32 octets de données :
    Délai d’attente de la demande dépassé.
    Délai d’attente de la demande dépassé.
    Délai d’attente de la demande dépassé.
    Délai d’attente de la demande dépassé.

    Statistiques Ping pour 162.168.112.1:
        Paquets : envoyés = 4, reçus = 0, perdus = 4 (perte 100%)

Nous faisons un Ping de la Vm 2 à Windows 2

[root@localhost poulet]# ping 192.168.102.2
    PING 192.168.102.2 (192.168.102.2) 56(84) bytes of data.
    64 bytes from 192.168.102.2: icmp_seq=1 ttl=128 time=0.210 ms
    64 bytes from 192.168.102.2: icmp_seq=2 ttl=128 time=0.625 ms
    64 bytes from 192.168.102.2: icmp_seq=3 ttl=128 time=0.607 ms
    64 bytes from 192.168.102.2: icmp_seq=4 ttl=128 time=0.673 ms
    64 bytes from 192.168.102.2: icmp_seq=5 ttl=128 time=0.718 ms
    64 bytes from 192.168.102.2: icmp_seq=6 ttl=128 time=0.651 ms
    64 bytes from 192.168.102.2: icmp_seq=7 ttl=128 time=0.672 ms
    ^C
    --- 192.168.102.2 ping statistics ---
    7 packets transmitted, 7 received, 0% packet loss, time 6003ms
    rtt min/avg/max/mdev = 0.210/0.593/0.718/0.162 ms

Nous faisons un Ping de la Vm 1 à Windows 1

[root@localhost it4]# ping 192.168.101.1
   PING 192.168.101.1 (192.168.101.1) 56(84) bytes of data.
   64 bytes from 192.168.101.1: icmp_seq=1 ttl=128 time=0.210 ms
   64 bytes from 192.168.101.1: icmp_seq=2 ttl=128 time=0.625 ms
   64 bytes from 192.168.101.1: icmp_seq=3 ttl=128 time=0.607 ms
   64 bytes from 192.168.101.1: icmp_seq=4 ttl=128 time=0.673 ms
   64 bytes from 192.168.101.1: icmp_seq=5 ttl=128 time=0.718 ms
   64 bytes from 192.168.101.1: icmp_seq=6 ttl=128 time=0.651 ms
   64 bytes from 192.168.101.1: icmp_seq=7 ttl=128 time=0.672 ms
   ^C
   --- 192.168.101.1 ping statistics ---
   7 packets transmitted, 7 received, 0% packet loss, time 6003ms
   rtt min/avg/max/mdev = 0.210/0.593/0.718/0.162 ms
  
## 2 Configuration du routage

Nous allons commencer par ajouter la route du PC1 vers le PC2 :

C:\Windows\ShaTaNuS>route add 192.168.102.0/24 mask 255.255.255.0 192.168.112.2
    OK!

Ensuite le PC2 ajoute la route vers le PC1 :

C:\WINDOWS\ShaTaNuS> route add 192.168.101.0/24 mask 255.255.255.0 192.168.112.1
    OK!

Maintenant au tour des Vm d'ajouter les routes : La VM1 qui ajoute les routes vers la VM2 et le PC2

[root@localhost it4]# ip route add 192.168.102.0/24 via 192.168.101.1 dev eth1
[root@localhost it4]# ip route add 192.168.112.0/24 via 192.168.101.1 dev eth1

Puis la VM2 qui ajoute les routes vers VM1 et le PC2

[root@localhost poulet]# ip route add 192.168.101.0/24 via 192.168.102.1 dev enp0s8
[root@localhost poulet]# ip route add 192.168.112.0/24 via 192.168.102.1 dev enp0s8

Maintenant nous allons ping tout ce beau monde

Le PC1 via le réseau 2 au PC2:

 C:\WINDOWS\ShaTaNuS> ping 192.168.102.1

    Envoi d’une requête 'Ping'  192.168.102.1 avec 32 octets de données :
    Réponse de 192.168.102.1 : octets=32 temps=1 ms TTL=54
    Réponse de 192.168.102.1 : octets=32 temps=1 ms TTL=54
    Réponse de 192.168.102.1 : octets=32 temps=1 ms TTL=54
    Réponse de 192.168.102.1 : octets=32 temps=1 ms TTL=54

    Statistiques Ping pour 192.168.102.1:
        Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
    Durée approximative des boucles en millisecondes :
        Minimum = 1ms, Maximum = 1ms, Moyenne = 1ms

Le PC2 via le réseau 1 au PC1:

C:\Windows\Simon>ping 192.168.101.1

    Envoi d’une requête 'Ping'  192.168.101.10 avec 32 octets de données :
    Réponse de 192.168.101.10 : octets=32 temps=1 ms TTL=63
    Réponse de 192.168.101.10 : octets=32 temps=1 ms TTL=63
    Réponse de 192.168.101.10 : octets=32 temps=1 ms TTL=63
    Réponse de 192.168.101.10 : octets=32 temps=1 ms TTL=63

    Statistiques Ping pour 192.168.101.10:
        Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
    Durée approximative des boucles en millisecondes :
        Minimum = 1ms, Maximum = 1ms, Moyenne = 1ms

La VM1 via le réseau 12 et 2 vers le PC2

[root@localhost it4]# pingpi 192.168.102.1
    PING 192.168.102.1 (192.168.102.1) 56(84) bytes of data.
    64 bytes from 192.168.102.1: icmp_seq=1 ttl=126 time=0.977 ms
    64 bytes from 192.168.102.1: icmp_seq=2 ttl=126 time=1.32 ms
    64 bytes from 192.168.102.1: icmp_seq=3 ttl=126 time=1.02 ms
    64 bytes from 192.168.102.1: icmp_seq=4 ttl=126 time=1.51 ms
    ^C
    
    --- 192.168.102.1 ping statistics ---
    4 packets transmitted, 4 received, 0% packet loss, time 3004ms
    rtt min/avg/max/mdev = 0.977/1.209/1.515/0.221 ms
    
    [root@localhost it4]# ping 192.168.112.2
    PING 192.168.112.2 (192.168.112.2) 56(84) bytes of data.
    64 bytes from 192.168.112.2: icmp_seq=1 ttl=127 time=1.27 ms
    64 bytes from 192.168.112.2: icmp_seq=2 ttl=127 time=1.47 ms
    64 bytes from 192.168.112.2: icmp_seq=3 ttl=127 time=1.52 ms
    64 bytes from 192.168.112.2: icmp_seq=4 ttl=127 time=1.60 ms
    ^C
    
    --- 192.168.112.2 ping statistics ---
    4 packets transmitted, 4 received, 0% packet loss, time 3005ms
    rtt min/avg/max/mdev = 1.278/1.469/1.605/0.128 ms

Et pour terminer la VM2 via le réseau 12 et 1 vers le PC1

[root@localhost poulet]# ping 192.168.112.1
    PING 192.168.112.1 (192.168.112.1) 56(84) bytes of data.
    64 bytes from 192.168.112.1: icmp_seq=1 ttl=127 time=1.39 ms
    64 bytes from 192.168.112.1: icmp_seq=2 ttl=127 time=1.82 ms
    64 bytes from 192.168.112.1: icmp_seq=3 ttl=127 time=1.86 ms
    64 bytes from 192.168.112.1: icmp_seq=4 ttl=127 time=1.13 ms
    
    [root@localhost poulet]# ping 192.168.101.1
    PING 192.168.101.1 (192.168.101.1) 56(84) bytes of data.
    64 bytes from 192.168.101.1: icmp_seq=1 ttl=126 time=1.10 ms
    64 bytes from 192.168.101.1: icmp_seq=2 ttl=126 time=1.19 ms
    64 bytes from 192.168.101.1: icmp_seq=3 ttl=126 time=1.11 ms
    64 bytes from 192.168.101.1: icmp_seq=4 ttl=126 time=1.13 ms