
# II. Lancement et configuration du lab

Config du router1.tp5.b1 :
```
10.0.0.0/24 is subnetted, 3 subnets
C       10.5.3.0 is directly connected, Ethernet0/1
S       10.5.2.0 [1/0] via 10.5.3.10
C       10.5.1.0 is directly connected, Ethernet0/0

```

Config du router2.tp5.b1 :
```
10.0.0.0/24 is subnetted, 3 subnets
C       10.5.3.0 is directly connected, Ethernet0/1
C       10.5.2.0 is directly connected, Ethernet0/0
S       10.5.1.0 [1/0] via 10.5.3.5
```

Config du server1.tp5.b1 :

```
route 10.5.2.0/24 via 10.5.1.254 dev enp0s3
hosts
10.5.2.10   client1 client1.tp5.b1
10.5.2.11   client2 client2.tp5.b1
```

Config du client1.tp5.b1 :

```
route 10.5.1.0/24 via 10.5.2.254 dev enp0s3
hosts
10.5.2.11   client2 client2.tp5.b1
10.5.1.10   server1 serveur1.tp5.b1
```

Config du client2.tp5.b1 :
```
route 10.5.1.0/24 via 10.5.2.254 dev enp0s3
hosts
10.5.2.10   client1 client1.tp5.b1
10.5.1.10   server1 serveur1.tp5.b1
```

Client1 ping server1 :

```
[root@client1 simon]$ ping server1
PING server1 (10.5.1.10) 56(84) bytes of data.
64 bytes from server1 (10.5.1.10): icmp_seq=1 ttl=62 time=30.2 ms
```

Client2 ping server1 :

```
[root@client2 simon]$ ping server1
PING server1 (10.5.1.10) 56(84) bytes of data.
64 bytes from server1 (10.5.1.10): icmp_seq=1 ttl=62 time=35.7 ms
```

# III. DHCP

## 1 Mise en palce du server DHCP

### 1 Renommer la machine

On entre dans le hosts :

```
nano /etc/hosts
dhcp-net2.tp5.b1
```

### 6 Faire un test

On a récupéré avec le DHCP une adresse random :
```
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:f6:d5:5b brd ff:ff:ff:ff:ff:ff
    inet 10.5.2.45/24 brd 10.5.2.255 scope global noprefixroute dynamic enp0s3
       valid_lft 493sec preferred_lft 493sec
    inet6 fe80::a00:27ff:fef6:d55b/64 scope link
       valid_lft forever preferred_lft forever
```