
**POSIX Sockets** (früher Berkeley Sockets genannt) sind ein im POSIX-Standard definiertes API zur Netzwerk-Kommunikation in Unix-Systemen.

Ein **Socket** ist ein logischer Endpunkt zur Netzwerk-Kommunikation eines Prozesses. Die Kommunikation erfolgt jeweils zwischen zwei Sockets.

# 1 Verwendung von POSIX Sockets  

Die Adress-Familie eines POSIX Sockets wird beim Aufruf von [socket()] festgelegt.

[bind()] ordnet dem Socket eine lokale Adresse zu, dies ist [für TCP‑ und UDP] - [Server] erforderlich.

Mit [accept()] blockiert der Server, bis ein Client eine [TCP]-Verbindung anfordert.

Zum Senden von Daten über einen Socket kann man [send() oder write()] verwenden.
# 2 Socket-Typen

POSIX definiert 3 verschiedene **Socket-Typen**, die unterschiedliche Kommunikationsmöglichkeiten bereitstellen:
- `SOCK_STREAM` entspricht einer TCP-Verbindung: bietet nach Verbindungsaufbau einen zuverlässigen bidirektionalen Bytestrom.
- `SOCK_DATAGRAM` entspricht dem zustandslosen UDP: einzelne Nachrichten begrenzter Länge (Datagramme) werden ohne Bestätigung und ohne Erfolgsgarantie übertragen.
- `SOCK_SEQPACKET` entspricht dem SCTP-Protokoll: ähnlich wie `SOCK_STREAM`, aber statt einem Bytestrom werden einzelne Nachrichten begrenzter Länge (_records_) jeweils komplett gesendet und empfangen.
    - SCTP（Stream Control Transmission Protocol），即**流媒体控制传输协议**，是一种可靠的基于无连接数据包网络如IP网络之上传输协议。 他被设计用来在IP网络上传输PSTN在窄带信令消息，同时也能支持宽带信令消息的传输

# 3 Socket-Adresse

Bevor man über einen Socket kommunizieren kann, muss man ihm eine **Socket-Adresse** zuweisen. POSIX definiert 3 **Adress-Familien**, die jeweils angeben, wie ein Socket adressiert wird und über welches Protokoll kommuniziert wird.

|Familie|Protokoll|Adresse besteht aus|
|---|---|---|
|`AF_INET`|IPv4|IP-Adresse, Port|
|`AF_INET6`|IPv6|IP-Adresse, Port, Scope-ID, Flow-Information|
|`AF_UNIX`|[Unix Domain Sockets](https://moodle.oncampus.de/modules/ir866/onmod/net/unix-sockets.html)|Pfad im Dateisystem (→ nächster Abschnitt)|


# 4 API-Funktionen

Das Socket-API besteht aus mehreren System Calls, dazu gehören unter anderem:

|   |   |
|---|---|
|`socket()`|erzeugt einen Socket und gibt einen _file descriptor_ darauf zurück|
|`bind()`|bindet einen (meist serverseitigen) Socket an eine Adresse|
|`listen()`|markiert einen gebundenen STREAM-Socket als passiv (_listening_)|
|`accept()`|wartet auf eine eingehende Verbindung auf einem _listening_ Socket|
|`connect()`|(clientseitig) baut aktiv eine TCP-Verbindung auf|
|`send()`|sendet Daten über einen Socket; alternativ: `write()`|
|`recv()`|empfängt Daten über einen Socket; alternativ: `read()`|
|`close()`|schließt den Socket, gibt Ressourcen frei|

Details zu den Funktionen finden Sie bei Bedarf in den entsprechenden Man Pages


# 5 TCP-Kommunikation

Für eine erfolgreiche Kommunikation müssen beide Seiten (Client und Server) die obigen System Calls in der korrekten Abfolge verwenden, die in der Grafik für TCP schematisch dargestellt ist:

TCP-Client und -Server mit Sockets
![](images/Pasted%20image%2020241220181623.png)

`accept()` blockiert den Server-Prozess, bis ein Client mit `connect()` eine Verbindung aufbaut, und gibt dann einen neuen Socket (im Bild: `c`) zurück, der an die offene Verbindung gebunden ist. Über diesen kann dann der Server Daten lesen und schreiben.

Sockets werden über spezielle _file descriptors_ identifiziert. Man kann wie bei Dateien mit `write()` und `read()` Daten schreiben und lesen. Die speziellen System Calls `send()` und `recv()` unterscheiden sich davon nur durch zusätzliche Optionen.

Ein Socket hat einen internen _send buffer_ für zu sendende und einen _receive buffer_ für empfangene Daten  (socket 有两个 buffer ). Im Unterschied zu normalen Datei-Operationen _blockieren_ Schreiben und Lesen auf einem Socket, wenn der entsprechende Puffer voll bzw. leer ist.


# 6 UDP-Kommunikation

Die Kommunikation über UDP ist programmtechnisch einfacher, da hier keine Verbindung aufgebaut werden muss:

UDP-Client und -Server mit Sockets
![](images/Pasted%20image%2020241220184606.png)

# 7 Socket-Informationen  (command ss )

Das Linux-Kommando `ss` (_socket statistics_) liefert detaillierte Informationen zu Sockets und aktiven Verbindungen im System. Es sollte mit root-Rechten verwendet werden, ansonsten erhalten Sie nur eingeschränkte Informationen.

Lesen Sie [ss(8)](https://man7.org/linux/man-pages/man8/ss.8.html) und probieren Sie das Kommando _als root_ mit verschiedenen Optionen aus. Finden Sie beispielsweise heraus
- welche TCP-Sockets auf eingehende Verbindungen warten und zu welchen Prozessen sie gehören
- welche UDP-Sockets auf Daten warten und zu zu welchen Prozessen sie gehören
- welche TCP-Verbindungen aktuell offen sind

In BSD-Systemen gibt es das ältere, weniger flexible `netstat`.

```
yzh@BNB23350:~$ ss
Netid      State        Recv-Q      Send-Q                  Local Address:Port                  Peer Address:Port            Process
u_str      ESTAB        0           0                                   * 19514                            * 25634
u_str      ESTAB        0           0                                   * 25631                            * 25632
u_str      ESTAB        0           0                                   * 23569                            * 23570
u_str      ESTAB        0           0                                   * 18467                            * 18466
u_str      ESTAB        0           0                                   * 25609                            * 25608
u_str      ESTAB        0           0                                   * 19510                            * 32773
u_str      ESTAB        0           0                /tmp/dbus-clzxK5Ywbg 25634                            * 19514
u_str      ESTAB        0           0                                   * 28675                            * 0
u_str      ESTAB        0           0                                   * 25608                            * 25609
u_str      ESTAB        0           0                                   * 18464                            * 18465
u_str      ESTAB        0           0                   /tmp/.X11-unix/X0 32773                            * 19510
u_str      ESTAB        0           0                                   * 29851                            * 0
u_str      ESTAB        0           0                                   * 18465                            * 18464
u_str      ESTAB        0           0                                   * 23570                            * 23569
u_str      ESTAB        0           0                                   * 18466                            * 18467
u_str      ESTAB        0           0                                   * 25632                            * 25631
v_str      ESTAB        0           0                                   *:4156552181                       2:50000
v_str      ESTAB        0           0                                   *:4156552182                       2:50000
v_str      ESTAB        0           0                                   *:4156552183                       2:50000
v_str      ESTAB        0           0                                   *:4156552184                       2:50001
v_str      ESTAB        0           0                                   *:4156552185                       2:50000
v_str      ESTAB        0           0                                   *:4156552187                       2:50001
v_str      ESTAB        0           0                                   *:4156552188                       2:50001
v_str      ESTAB        0           0                                   *:4156552189                       2:50001
v_str      ESTAB        0           0                                   *:4156552190                       2:50000
v_str      ESTAB        0           0                                   *:4156552191                       2:50000
v_str      ESTAB        0           0                                   *:4156552192                       2:50003
v_str      ESTAB        0           0                                   *:4156552194                       2:50003
v_str      ESTAB        0           0                                   *:1                                2:4024209514
v_str      ESTAB        0           0                                   *:4156552193                       2:4024208623
v_str      ESTAB        0           0                                   *:4156552198                       2:4024208632
v_str      ESTAB        0           0                                   *:4156552198                       2:4024208631
v_str      ESTAB        0           0                                   *:4156552198                       2:4024208630
v_str      ESTAB        0           0                                   *:4156552198                       2:4024208629
v_str      ESTAB        0           0                                   *:4156552198                       2:4024208628
v_str      ESTAB        0           0                                   *:4156552196                       2:4024208626
v_str      ESTAB        0           0                                   *:4156552197                       2:4024208627
v_str      ESTAB        0           0                                   *:4156552200                       2:4024209507
v_str      ESTAB        0           0                                   *:4156552200                       2:4024209506
v_str      ESTAB        0           0                                   *:4156552200                       2:4024209505
v_str      CLOSING      0           0                                   *:4156552200                       2:4024209504
```
