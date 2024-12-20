
Ausgehende IP-Pakete werden vom Kernel nach dem bekannten Prinzip des [_longest prefix match_](https://de.wikipedia.org/wiki/Longest_Prefix_Match) geroutet. Dazu gibt es im Kernel zwei getrennte **Routing-Tabellen** für IPv4 und IPv6.


Die Einträge dieser Tabellen kann man in Linux mit dem Subkommando `ip route` anzeigen und modifizieren. In BSD- und älteren Linux-Systemen gibt es stattdessen die Befehle `netstat -r` und `route`.

Sehen Sie sich die Routing-Tabellen Ihres Ubuntu-Systems an:
- `ip route` für IPv4
- `ip -6 route` für IPv6

Was ist jeweils die Default-Route?

Die **Default-Route** wird für Verbindungen außerhalb der bekannten lokalen Netze verwendet. Wenn eine IP-Adresse per DHCP konfiguriert wird, trägt der DHCP-Client normalerweise automatisch eine passende Default-Route ein.

Jede Route hat eine _Metrik_. Falls mehrere mögliche Routen zum Ziel existieren, wird die mit der kleinsten Metrik verwendet.

Mit dem Kommando `ip route get <address>` können Sie ausprobieren, wie Pakete zu einer gegebenen Zieladresse geroutet werden; das ist beim Debuggen der Routing-Tabellen hilfreich.


# 1 Routing table 理论 


![](images/Pasted%20image%2020241220161915.png)



Example
Routing Table

|Prefix|Next Hop|
|---|---|
|`192.168.0.0/16`|Router A|
|`192.168.1.0/24`|Router B|
|`10.0.0.0/8`|Router C|

Incoming Packet
Destination IP: `192.168.1.5`

Matching Process
1. Compare `192.168.1.5` with `192.168.0.0/16`.
    - Matches: First 16 bits are the same.
2. Compare `192.168.1.5` with `192.168.1.0/24`.
    - Matches: First 24 bits are the same.
3. The longest matching prefix is `192.168.1.0/24` (24 bits), so the packet is forwarded to **Router B**.




# 2 相关命令 

## 2.1 ip route: 查看 相关的 route table 中的条换 

```sh
yzh@BNB23350:~$ ip route
default via 172.31.144.1 dev eth0 proto kernel
172.31.144.0/20 dev eth0 proto kernel scope link src 172.31.144.86

yzh@BNB23350:~$ ip -6 route
fe80::/64 dev eth0 proto kernel metric 256 pref medium
```

Bedeutung dieser Konfiguration
1. **Standardroute:**    
    - Alle Pakete, die nicht für das Subnetz `172.31.144.0/20` (其实就是 ziel Subnetz )  bestimmt sind, werden an den Gateway `172.31.144.1` weitergeleitet.
    - Diese Route ermöglicht Verbindungen ins Internet oder zu anderen Netzwerken.
2. **Netzwerkroute:**
    - Pakete, die für das lokale Subnetz `172.31.144.0/20` (其实就是 ziel Subnetz )  bestimmt sind, werden direkt über `eth0` gesendet, ohne den Gateway zu passieren.

default via 172.31.144.1 dev eth0 proto kernel
- **`default`**: Diese Route (就是 这一行 这个Entry)  wird verwendet, wenn keine spezifischere Route für eine Ziel-IP-Adresse gefunden wird.
- **`via 172.31.144.1`**: Der Gateway (nächster Hop) für diese Route ist `172.31.144.1`.
- **`dev eth0`**: Das Netzwerkinterface, das für diese Route verwendet wird, ist `eth0`.
- **`proto kernel`**: Diese Route wurde automatisch vom Kernel hinzugefügt (z. B. durch DHCP oder eine statische Konfiguration).


172.31.144.0/20 dev eth0 proto kernel scope link src 172.31.144.86
- **`172.31.144.0/20`**: Diese Route (就是 这一行 这个Entry) deckt alle IP-Adressen im Subnetz `172.31.144.0` mit der Subnetzmaske `/20` (255.255.240.0) ab.
    - **Adressbereich**: Von `172.31.144.0` bis `172.31.159.255`.
- **`dev eth0`**: Datenpakete für dieses Subnetz werden über das Interface `eth0` gesendet.
- **`proto kernel`**: Diese Route wurde ebenfalls vom Kernel hinzugefügt.
- **`scope link`**: Diese Route ist auf direkt angeschlossene Geräte im lokalen Netzwerk beschränkt.
- **`src 172.31.144.86`**: Die Quelladresse für Pakete, die über diese Route gesendet werden, ist `172.31.144.86`.
    - The **`src 192.168.1.100`** in routing information specifies the **source IP address** that will be used when the system sends a packet via the corresponding route.
    - 就是说 这个 如果要使用这个Route Entry,  Package就会从 这个  source-IP-address 发出去, 发到 next Hop (就是一个 gateway )


## 2.2 `ip route get <zieladdress>` 

wie Pakete zu einer gegebenen Zieladresse geroutet werden;

```
ip route get 8.8.8.8

ausgabe
8.8.8.8 via 192.168.1.1 dev eth0 src 192.168.1.100 uid 1000
    cache
```

- **`via 192.168.1.1`**: Nächster Hop (Gateway).
- **`dev eth0`**: Schnittstelle, die für die Route verwendet wird.
- **`src 192.168.1.100`**: Quell-IP-Adresse des Pakets.
- **`cache`**: Zeigt an, dass die Route zwischengespeichert ist.


## 2.3 ip route add/del

添加 删除 Routing table 中的 Route 

例子
1 
Suppose `eth0` has two IP addresses: `192.168.1.100` and `192.168.1.101`.  
The routing table determines which IP address to use for a given route.

2 
```
ip address add 192.168.1.101 dev eth0
ip route add 10.0.0.0/24 via 192.168.1.1 src 192.168.1.101
```

- Packets to the `10.0.0.0/24` subnet will use the source IP `192.168.1.101`.
- Packets to other destinations (e.g., the default route) will use the source IP `192.168.1.100`.

3 
Checking the Source Address
You can verify which source IP will be used for a specific destination using the following command:
`ip route get 8.8.8.8`

Example Output:
`8.8.8.8 via 192.168.1.1 dev eth0 src 192.168.1.100`

