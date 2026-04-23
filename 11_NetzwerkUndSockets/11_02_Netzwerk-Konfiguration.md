
Das Netzwerk muss vor der Benutzung geeignet konfiguriert werden. Dazu sind root-Rechte nötig.

# 1 Interface-Konfiguration  (ip command )

Je nach Typ einer Netzwerkschnittstelle können verschiedene Parameter konfiguriert werden. Dies erfolgt unter anderem mit dem System Call `ioctl()` auf einem für die Schnittstelle geöffneten [Socket](https://moodle.oncampus.de/modules/ir866/onmod/net/sockets.html).

In der Praxis verwendet man aber nicht direkt solche System Calls, sondern spezielle Kommandos:
- `ip` ist das Standard-Kommando zur Interface-Konfiguration in modernen Linux-Systemen. Es hat zahlreiche Subkommandos, siehe [ip(8)](https://man7.org/linux/man-pages/man8/ip.8.html).
- Das ältere `ifconfig` wird in Systemen der BSD-Familie und älteren Linux-Systemen verwendet. Es wird in dieser Einheit nicht weiter betrachtet.


## 1.1 **IP-Adressen** (ip address command )

**IP-Adressen** sind die wichtigsten Schnittstellen-Parameter: Eine Netzwerkschnittstelle kann mehrere zugewiesene IP-Adressen haben. Der TCP/IP-Protokollstack des Kernels unterstützt gleichzeitig IPv4 und IPv6 (_dual stack_). IP-Adressen werden mit dem Subkommando `ip address` angezeigt und geändert.

Lesen Sie zunächst ip-address(8) und probieren Sie das Kommando in Ubuntu aus:
- Sehen Sie sich die IP-Adressen Ihrer Ubuntu-VM an: `ip address`
- Fügen Sie mit dem Kommando `ip address add` zum Ethernet-Interface die Adresse 172.16.3.3/24 hinzu.
- Jetzt sollte die zusätzliche Adresse mit `ip address` angezeigt werden.
- Entfernen Sie die Adresse mit `ip address del` wieder.

- ip address
- ip address add
    - `sudo ip address add 172.16.3.3/24 dev eth0`
    - **`172.16.3.3/24`**: Die neue IP-Adresse mit der Subnetzmaske.
    - **`dev eth0`**: Das Ziel-Interface (`eth0` ist ein Beispiel; passen Sie es bei Bedarf an).
- ip address del
    - `sudo ip address del 172.16.3.3/24 dev eth0`

Hinweise
- **Root-Rechte erforderlich**: Für Änderungen an den IP-Adressen benötigen Sie `sudo`.
- **Temporäre Änderungen**: Änderungen mit `ip address add` oder `ip address del` sind nicht persistent. Nach einem Neustart werden sie zurückgesetzt. Um Änderungen dauerhaft zu machen, bearbeiten Sie die Netzwerkkonfiguration in `/etc/netplan/` oder mit einem anderen Netzwerkmanager.

---


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

yzh@BNB23350:~$ sudo ip address add 172.16.3.3/24 dev eth0
[sudo] password for yzh:
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
    inet 172.16.3.3/24 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::215:5dff:fe4c:269c/64 scope link
       valid_lft forever preferred_lft forever
yzh@BNB23350:~$ sudo ip address del 172.16.3.3/24 dev eth0
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


## 1.2 MAC-Adressen Konfigurieren( `ip link command` )


IP-Adressen werden oft automatisch über das Protokoll [**DHCP**](https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol) konfiguriert. Dazu muss auf dem Rechner ein entsprechender DHCP-Client-Prozess als Service laufen.
(Dynamic Host Configuration Protocol:  host dynamically obtains IP address from network server when it "joins" network )

Das Subkommando `ip link` zeigt alle vorhandenen Netzwerkschnittstellen mit MAC-Adressen und aktuellem Status an. Mit `ip link set` können verschiedene Schnittstellen-Parameter geändert werden und die Schnittstelle aktiviert oder deaktiviert werden.

- Sehen Sie sich die Netzwerkschnittstellen Ihrer Ubuntu-VM an: `ip link`
- Deaktivieren Sie das (virtuelle) Ethernet-Interface: `ip link set <interface> down` und sehen Sie sich mit `ip link` den geänderten Status an. Verifizieren Sie, dass keine Netzwerkverbindungen mehr möglich sind.
- Aktivieren Sie das Interface wieder: `ip link set <interface> up`.

- sudo ip link set eth0 down
    -  the ping www.google.com 
- sudo ip link set eth0 up

Hinweise:
- Ersetzen Sie `eth0` durch den Namen Ihrer Netzwerkschnittstelle, wenn dieser anders lautet (siehe die Ausgabe von `ip link`).
- Die Änderungen mit `ip link set` sind **temporär** und gehen nach einem Neustart verloren. Um die Änderungen dauerhaft zu machen, bearbeiten Sie die Netzwerkkonfiguration in `/etc/netplan/` oder verwenden Sie Ihren Netzwerkmanager.

```sh
yzh@BNB23350:~$ ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:15:5d:4c:26:9c brd ff:ff:ff:ff:ff:ff

```

----

In Unix kann man auch die MAC-Adresse eines physischen Netzwerkadapters konfigurieren, wenn die Hardware das unterstützt. Die MAC-Adresse der Hardware dient als Default-Wert beim Booten.

Eine Filterung nach MAC-Adressen ist daher als Sicherheitsmaßnahme wirkungslos. Auch DHCP-Server, die anhand der MAC-Adressen feste IP-Adressen an Clients vergeben, kann man so leicht überlisten.

Die Verschlüsselung von WLAN-Verbindungen wird über spezielle Tools konfiguriert, z.B. [wpa_supplicant](https://wiki.archlinux.org/title/wpa_supplicant).

# 2 Hostname (hostname command )

Der **Hostname** des Rechners wird mit dem Kommando `hostname` abgefragt und geändert. Ein statischer Hostname kann in einer Konfigurationsdatei gesetzt werden, z.B. bei Linux in `/etc/hostname`.

# 3 DNS-Konfiguration (`/etc/resolv.conf`)

Es gibt in Unix verschiedene Mechanismen zur DNS-Auflösung von Hostnamen, auf die wir hier nicht im Detail eingehen. Dazu muss die IP-Adresse von mindestens einem DNS-Server bekannt sein. DNS-Server-Adressen können statisch konfiguriert oder dynamisch von einem DHCP-Server geliefert werden.

Je nach verwendetem _DNS resolver_ werden die DNS-Server in unterschiedliche Konfigurationsdateien eingetragen, z.B. ==`/etc/resolv.conf` ==für den _resolver_ der C-Systembibliothek.

# 4 Network Management Tools

Es wäre recht mühsam, Netzwerkschnittstellen immer manuell oder über ein init-Script zu konfigurieren.

Daher gibt es verschiedene _network management tools_, die das Netzwerk beim Booten anhand von Konfigurationsdateien konfigurieren und zum Teil auch dynamische Zustandsänderungen wie ein neu eingestecktes Ethernet-Kabel oder den Wechsel des WLAN-Netzwerks automatisch behandeln können. Für manche dieser Tools gibt es ein grafisches Frontend.

Beispiele solcher Tools für Linux sind [NetworkManager](https://en.wikipedia.org/wiki/NetworkManager) (Default bei Ubuntu Desktop), [netplan](https://netplan.io/) und [systemd-networkd](https://wiki.archlinux.org/title/systemd-networkd). Auf die Details gehen wir hier nicht weiter ein.



