
> Eine **Netzwerkschnittstelle (network interface)** ist logischer Zugangspunkt für Netzwerkverbindungen.

Netzwerkschnittstellen werden in Unix vom Kernel verwaltet. ==Es gibt für sie keine entsprechenden Gerätedateien in `/dev`. Jede Netzwerkschnittstelle hat einen eindeutigen Namen, dessen Präfix typischerweise vom Interface-Typ abhängt.==

Es gibt verschiedene Typen von Netzwerkschnittstellen. Manche davon entsprechen **physischer Netzwerk-Hardware**, z.B. Ethernet, Wifi, Bluetooth. Andere sind **virtuelle Schnittstellen**, die den Datenverkehr letztendlich an ein anderes Interface weiterleiten, z.B. für VLANs, Bridges, VPNs oder virtuelle Netzwerke.

Wie gehen hier nur auf die drei wichtigsten Interface-Typen ein:
- ==**Loopback Interface** (Name `lo`)== : diese auf jedem Unix-System vorhandene virtuelle Schnittstelle erlaubt nur Verbindungen zwischen Prozessen des lokalen Rechners und ist von der Außenwelt isoliert. Sie hat die speziellen IP-Adressen `127.0.0.1` und `::1`  (IPv6 格式) und den symbolischen Hostnamen `localhost`.
- **Ethernet Interface** (Name meist `en*` oder `eth*`): repräsentiert einen Ethernet-Adapter.
- **WLAN Interface** (Name meist `wl*`): repräsentiert einen WLAN-Adapter.

Sehen Sie nach, welche Netzwerkschnittstellen es auf Ihrem Ubuntu-System gibt: `ip link`

```sh
yzh@BNB23350:~$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:4c:26:9c brd ff:ff:ff:ff:ff:ff
```


```sh
yzh@BNB23350:~$ ip address
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
       
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 00:15:5d:4c:26:9c brd ff:ff:ff:ff:ff:ff
    inet 172.31.144.86/20 brd 172.31.159.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::215:5dff:fe4c:269c/64 scope link
       valid_lft forever preferred_lft forever

```










