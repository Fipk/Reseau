
# Lab 2 : Un peu de complexité (et d'utilité ?...)

## 2. Mise en place du lab

Pour vérifier que tout fonctionne correctement et qu'on a tout bien configuré:

`show ip route`
```
r1#show ip route
     10.0.0.0/8 is variably subnetted, 7 subnets, 2 masks
O       10.6.100.8/30 [110/20] via 10.6.100.2, 00:07:50, Ethernet0/0
O       10.6.100.12/30 [110/20] via 10.6.100.6, 00:07:50, Ethernet0/1
C       10.6.100.0/30 is directly connected, Ethernet0/0
O       10.6.101.0/30 [110/30] via 10.6.100.6, 00:07:50, Ethernet0/1
                      [110/30] via 10.6.100.2, 00:07:50, Ethernet0/0
C       10.6.100.4/30 is directly connected, Ethernet0/1
O       10.6.201.0/24 [110/40] via 10.6.100.6, 00:07:50, Ethernet0/1
                      [110/40] via 10.6.100.2, 00:07:51, Ethernet0/0
C       10.6.202.0/24 is directly connected, Ethernet0/2
```

ou alors:

`show ip ospf neighbor`

```
r3#show ip ospf neighbor
Neighbor ID     Pri   State           Dead Time   Address         Interface
5.5.5.5           1   FULL/BDR        00:00:32    10.6.101.2      Ethernet0/2
4.4.4.4           1   FULL/BDR        00:00:35    10.6.100.14     Ethernet0/1
2.2.2.2           1   FULL/BDR        00:00:32    10.6.100.9      Ethernet0/0
```

On peut aussi essayer de ping entre server et client:

```
[root@client1 simon]$ ping server1
PING server1 (10.6.202.10) 56(84) bytes of data.
64 bytes from server1 (10.6.202.10): icmp_seq=1 ttl=60 time=59.4 ms
64 bytes from server1 (10.6.202.10): icmp_seq=2 ttl=60 time=60.1 ms
```

# Lab 3 : Let's end this properly

##1. NAT: accès internet

On recupére beaucoup d'html (je n'ai pas tout mis):

```
r2#telnet trip-hop.net 80
Trying trip-hop.net (213.186.33.4, 80)... Open
GET /
HTTP/1.1 404 Not Found
Set-Cookie: 240planBAK=R2339298881; path=/; expires=Tue, 26-Feb-2019 12:48:15 GMT
Server: nginx
Date: Tue, 26 Feb 2019 11:48:10 GMT
Content-Type: text/html; charset=utf8
Pragma: no-cache
Cache-Control: no-store, no-cache, must-revalidate, post-check=0, pre-check=0
Content-Length: 5329
X-IPLB-Instance: 17298
X-Cache: MISS from PF1-BOR1FR
X-Cache-Lookup: MISS from PF1-BOR1FR:3128
Connection: close

<html xml:lang="en-GB" lang="en-GB">
<head>
<meta name="viewport" content="width=device-width">
<title qtlid="283606">Site not installed</title>
            <meta http-equiv="refresh" content="480">
<meta http-equiv="Content-Type" content="text/html; charset=utf-8">
```

##2. Un service d'infra

On obtient ceci en faisant curl server1:

```
[root@client1 simon]# curl server1
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">

<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">
    <head>
        <title>test</title>
    </head>
</html>
```