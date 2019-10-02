 # I.Gather informations

Pour récupérer la liste des cartes réseau avec leur nom, leur ip et leur adresse MAC on écrit:

`ip a`

```
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:9b:a0:90 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
       valid_lft 85949sec preferred_lft 85949sec
    inet6 fe80::440f:bfe1:9d20:6c2c/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:d1:2e:c0 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.105/24 brd 192.168.56.255 scope global dynamic noprefixroute enp0s8
       valid_lft 749sec preferred_lft 749sec
    inet6 fe80::3fa5:6a63:8973:cf74/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
```

La carte réseau enp0s8 a bien récupérer une IP en DHCP car l'on peut se connecter en SSH alors que rien n'apparait dessus:

`sudo nmcli -f DHCP4 con show enp0s8`

```
enp0s8 - no such connection profile.
```

`sudo nmcli -f DHCP4 con show enp0s3`

```
DHCP4.OPTION[1]:                        domain_name = lan
DHCP4.OPTION[2]:                        domain_name_servers = 192.168.1.254
DHCP4.OPTION[3]:                        expiry = 1570138639
DHCP4.OPTION[4]:                        ip_address = 10.0.2.15
DHCP4.OPTION[5]:                        requested_broadcast_address = 1
DHCP4.OPTION[6]:                        requested_dhcp_server_identifier = 1
DHCP4.OPTION[7]:                        requested_domain_name = 1
DHCP4.OPTION[8]:                        requested_domain_name_servers = 1
DHCP4.OPTION[9]:                        requested_domain_search = 1
DHCP4.OPTION[10]:                       requested_host_name = 1
DHCP4.OPTION[11]:                       requested_interface_mtu = 1
DHCP4.OPTION[12]:                       requested_ms_classless_static_routes = 1
DHCP4.OPTION[13]:                       requested_nis_domain = 1
DHCP4.OPTION[14]:                       requested_nis_servers = 1
DHCP4.OPTION[15]:                       requested_ntp_servers = 1
DHCP4.OPTION[16]:                       requested_rfc3442_classless_static_routes = 1
DHCP4.OPTION[17]:                       requested_routers = 1
DHCP4.OPTION[18]:                       requested_static_routes = 1
DHCP4.OPTION[19]:                       requested_subnet_mask = 1
DHCP4.OPTION[20]:                       requested_time_offset = 1
DHCP4.OPTION[21]:                       requested_wpad = 1
DHCP4.OPTION[22]:                       routers = 10.0.2.2
DHCP4.OPTION[23]:                       subnet_mask = 255.255.255.0
```

Pour afficher la table ARP on écrit:

`ip route`

```
default via 10.0.2.2 dev enp0s3 proto dhcp metric 100
10.0.2.0/24 dev enp0s3 proto kernel scope link src 10.0.2.15 metric 100
192.168.56.0/24 dev enp0s8 proto kernel scope link src 192.168.56.105 metric 101
```
Cette route est vers le réseau enp0s3 elle est utilisée pour une connexion locale, la passerelle de cette route est à l'ip 10.0.2.2 et cette IP est portée par 10.0.2.15.

Cette route est vers le réseau enp0s8 elle est utilisée pour une connexion locale, la passerelle de cette route est à l'ip 192.168.56.1 et cette IP est portée par 192.168.56.105.

Pour afficher la table de routage on écrit:

`ip neigh show`

```
10.0.2.2 dev enp0s3 lladdr 52:54:00:12:35:02 REACHABLE
192.168.56.100 dev enp0s8 lladdr 08:00:27:ec:0c:31 STALE
192.168.56.1 dev enp0s8 lladdr 0a:00:27:00:00:07 REACHABLE
```

Pour récupérer la liste des ports en écoute sur la machine en TCP et UDP on écrit:

`ss -lntup`

```
Netid      State        Recv-Q       Send-Q                      Local Address:Port             Peer Address:Port
udp        UNCONN       0            0                               127.0.0.1:323                   0.0.0.0:*
udp        UNCONN       0            0                   192.168.56.105%enp0s8:68                    0.0.0.0:*
udp        UNCONN       0            0                        10.0.2.15%enp0s3:68                    0.0.0.0:*
udp        UNCONN       0            0                                   [::1]:323                      [::]:*
tcp        LISTEN       0            128                               0.0.0.0:22                    0.0.0.0:*
tcp        LISTEN       0            128                                  [::]:22                       [::]:*
```

pour afficher l'état actuel du firewall on tape:

`firewall-cmd --list-all`

```
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources:
  services: cockpit dhcpv6-client ssh
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

 # II.Edit configuration

 ## 1.Configuration cartes réseau

 On va modifier la carte réseau privée pour ce faire on doit changer le fichier enp0s8:

 `vim /etc/sysconfig/network-scripts//ifcfg-enp0s8`

 ```
 BOOTPROTO=static
NAME=enp0s8
DEVICE=enp0s8
ONBOOT=yes

NETMASK=255.255.255.0
IPADDR=192.168.56.101
 ```

 On créer ensuite un deuxieme reseau host only:

 `vim /etc/sysconfig/network-scripts//ifcfg-enp0s9`

  ```
 BOOTPROTO=static
NAME=enp0s9
DEVICE=enp0s9
ONBOOT=yes

NETMASK=255.255.255.0
IPADDR=192.168.57.101
 ```

 Ensuite on restart l'interface et en faisant:

 `ip a`

 ```
 1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:9b:a0:90 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
       valid_lft 86004sec preferred_lft 86004sec
    inet6 fe80::440f:bfe1:9d20:6c2c/64 scope link noprefixroute
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:d1:2e:c0 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.101/24 brd 192.168.56.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fed1:2ec0/64 scope link
       valid_lft forever preferred_lft forever
4: enp0s9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:91:9b:33 brd ff:ff:ff:ff:ff:ff
    inet 192.168.57.101/24 brd 192.168.57.255 scope global noprefixroute enp0s9
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe91:9b33/64 scope link
       valid_lft forever preferred_lft forever
 ```