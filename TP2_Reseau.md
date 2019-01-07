 # I.Exploration locale en solo

## 1.Affichage d'informations sur la pile TCP/IP locale

Pour afficher les infos des cartes réseaux de notre PC:

Il faut taper ipconfig /all dans le powershell.

Carte réseau sans fil Wi-Fi 2 :
***
Suffixe DNS propre à la connexion. . . : auvence.co

Description. . . . . . . . . . . . . . : Killer Wireless-n/a/ac 1535 Wireless 
Network Adapter #2

Adresse physique . . . . . . . . . . . : 9C-B6-D0-1A-59-D7  
DHCP activé. . . . . . . . . . . . . . : Oui  
Configuration automatique activée. . . : Oui  
Adresse IPv6 de liaison locale. . . . .: fe80::9ce5:2fed:50e9:9432%20(préféré)  
Adresse IPv4. . . . . . . . . . . . . .: 10.33.2.72(préféré)  
Masque de sous-réseau. . . . . . . . . : 255.255.252.0  
Bail obtenu. . . . . . . . . . . . . . : mardi 11 décembre 2018 09:12:03  
Bail expirant. . . . . . . . . . . . . : mardi 11 décembre 2018 11:32:06  
Passerelle par défaut. . . . . . . . . : 10.33.3.253  
Serveur DHCP . . . . . . . . . . . . . : 10.33.3.254  
IAID DHCPv6 . . . . . . . . . . . : 194819792  
DUID de client DHCPv6. . . . . . . . : 00-01-00-01-22-61-8B-8A-E0-D5-5E-21-38-DD  
 Serveurs DNS. . .  . . . . . . . . . . : 10.0.10.20  
                                    8.8.8.8 
 NetBIOS sur Tcpip. . . . . . . . . . . : Activé  

Le nom est "Carte réseau sans fil Wi-Fi 2" pour l'interface WiFi.
Son adresse Mac est 9C-B6-D0-1A-59-D7.
Son ip correspond à 10.33.2.72/22
L'adresse réseau est 10.33.0.0 et l'adresse de broadcast est 10.33.3.255
***
Carte Ethernet Ethernet 2 :

   Statut du média. . . . . . . . . . . . : Média déconnecté  
   Suffixe DNS propre à la connexion. . . : lan  
   Description. . . . . . . . . . . . . . : Killer E2500 Gigabit Ethernet Controller #2  
   Adresse physique . . . . . . . . . . . : E0-D5-5E-21-38-DD  
   DHCP activé. . . . . . . . . . . . . . : Oui  
   Configuration automatique activée. . . : Oui 

Le nom est "Carte Ethernet Ethernet 2" pour l'interface WiFi.
Son adresse Mac est E0-D5-5E-21-38-DD.
Il n'a pas d'ip, ainsi que l'adresse de passerelle et d'adresse réseau.

### EN graphique


Pour trouver l'interface graphique il suffit d'aller dans:
- Paramètres;
- Réseau et internet;
- Afficher vos propriétés réseau.

Nous retrouverons ensuite ce que nous avons eu dans la ligne de commande comme écrit au-dessus.

La gateway dans le réseau Ingésup permet de faire la liaison entre deux réseaux pour éxécuter de sprotocoles.

## Modification des informations

### A Modification d'adresse IP.

La première adresse ip du réseau est 10.33.0.1/22 et la dernière 10.33.3.252/22

Pour changez l'adresse ip il faut aller dans:
- panneau de configuration;
- Réseau et Internet;
- Centre réseau et partage;
- Modifier les paramètres de la carte;
- Double clic sur la carte WiFi ou Ethernet;
- Propriétés;
- Protocole Internet version 4 (TCP/IPv4);
- Cocher utiliser l'adresse IP suivante.

### B nmap

Pour changez l'adresse ip en vérifiant avec nmap une adresse ip libre il faut rentrer en ligne de commande:

nmap -sn -PE 10.33.2.200

Ensuite si on obtient:

Starting Nmap 7.70 ( https://nmap.org ) at 2018-12-11 12:02 Paris, Madrid  
Note: Host seems down. If it is really up, but blocking our ping probes, try -Pn  
Nmap done: 1 IP address (0 hosts up) scanned in 2.04 seconds  

On tape dans en ligne de commande:

nmap -Pn 10.33.2.200

Ainsi on obtient : 

Starting Nmap 7.70 ( https://nmap.org ) at 2018-12-11 12:05 Paris, Madrid  
Nmap done: 1 IP address (0 hosts up) scanned in 2.00 seconds

On peut donc changer son adresse ip avec l'dresse ip 10.33.2.200/22 avec comme passerelle 10.33.3.253 et masque 255.255.252.0.

En changeant l'adresse de gateway, je n'ai pu accès à internet.

## Exploration en locale en duo

On a tout d'abord branché les cable RJ45 reliant les 2 PC. Puis nous avons désactiver notre firewall Nous avons ensuite changer notre adresse IP pour créer un réseau : Le premier PC a donc l'adresse IP : 192.168.137.1 et le deuxième : 192.168.137.16 avec un masque de sous-réseau en /24 nous avons effectué un ping chacun :

Envoi d’une requête 'Ping'  192.168.137.1 avec 32 octets de données :
Réponse de 192.168.137.1 : octets=32 temps=2 ms TTL=128
Réponse de 192.168.137.1 : octets=32 temps=2 ms TTL=128
Réponse de 192.168.137.1 : octets=32 temps=2 ms TTL=128
Réponse de 192.168.137.1 : octets=32 temps=2 ms TTL=128

## Petit chat privé

On a telechargé Netcat puis on a exécuté les deux commandes suivante :

nc.exe -l -p 8888 pour écouter sur le port 8888 en tant que "serveur" et le deuxième à fait la commande nc.exe 
192.168.137.1 8888 et nous avons réussi à communiquer entre nous.

J:\OneDrive - Ynov\Ynov\Reseau\netcat-1.11>nc.exe 192.168.137.1 8888
coucou
Hello
C'est marrant ca marche 
Hello World

##  Wireshark

