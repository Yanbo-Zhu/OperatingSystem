
Unix 系统内部通讯 使用 Unix Domain Sockets . 他的地址 是 对应的文件的地址, 不是 portnummer 

**Unix Domain Sockets** (kurz: Unix Sockets) sind ein in POSIX standardisierter Mechanismus zur Interprozess-Kommunikation. Es handelt sich um Sockets mit der Adressfamilie `AF_UNIX`. Als Adresse dient dabei ein Pfad im Dateisystem; Portnummern gibt es nicht.

Das API ist analog zu den vorher behandelten Internet-Sockets. Es sind auch die gleichen 3 Socket-Typen möglich: `SOCK_STREAM`, `SOCK_DGRAM`, `SOCK_SEQPACKET`.


# 1 Benannte Sockets

产生一个 Unix Domain Socket  会产生一个对应文件  在文件系统里面
Ein benannter Unix Domain Socket wird im Dateisystem als spezielle Datei vom Typ _socket_ (`s` im Output von `ls -l`) erzeugt. Der Zugriff darauf kann durch die üblichen Dateizugriffsrechte beschränkt werden.

Ein einmal erzeugter benannter Unix Domain Socket bleibt unabhängig vom erzeugenden Prozess persistent im Dateisystem erhalten, bis er explizit mit `unlink()` gelöscht wird.

Bei Unix Domain Sockets wird der Begriff "Socket" mit zwei verschiedenen Bedeutungen verwendet:
1. Die Socket-Datei, deren Pfad als Adresse verwendet wird — diese gibt es nur einmal.
2. Der Socket im Programm, der an eine Socket-Datei gebunden wird — davon gibt es auf jeder Seite der Kommunikation einen.


# 2 Unix Domain Sockets 的具体使用  ( `ss -lxp`)

> Unix Domain Sockets werden oft als Schnittstelle für lokale Services verwendet: Ein Service-Prozess "hört" auf einem Unix Domain Socket unter einem bekannten Pfad. Clients bauen eine Verbindung zu diesem Socket auf und senden darüber Anforderungen.

Typische Beispiele dafür in Ubuntu sind
- Logging-Messages werden an den Socket `/run/systemd/journal/dev-log` gesendet.
- `ssh` kommuniziert zur Authentifizierung ausgehender Verbindungen mit dem [ssh-agent](https://man7.org/linux/man-pages/man1/ssh-agent.1.html) des Users über den Socket `./tmp/ssh-*/agent.<pid>`
- Der Wayland-Compositor für die grafische Oberfläche ist über den Socket `/run/user/<uid>/wayland-0` erreichbar.
- Der Pulseaudio Sound Server des Users ist über `/run/usr/<uid>/pulse/native` erreichbar.

Sehen Sie mit `ss -lxp` an, welche Prozesse auf welchen Unix Domain Sockets hören.

```
yzh@BNB23350:~$ ss -lxp
Netid      State       Recv-Q      Send-Q                                Local Address:Port            Peer Address:Port     Process
u_str      LISTEN      0           4096                             /run/WSL/1_interop 25615                      * 0
u_str      LISTEN      0           4096                             /run/WSL/1_interop 24587                      * 0
u_str      LISTEN      0           4096                             /run/WSL/9_interop 26631                      * 0
u_seq      LISTEN      0           1                      /mnt/wslg/weston-notify.sock 20490                      * 0
u_str      LISTEN      0           4096                /var/run/dbus/system_bus_socket 23566                      * 0
u_str      LISTEN      0           128                 /mnt/wslg/runtime-dir/wayland-0 18456                      * 0
u_str      LISTEN      0           1                                 /tmp/.X11-unix/X0 18457                      * 0
u_str      LISTEN      0           5                /mnt/wslg/runtime-dir/pulse/native 19507                      * 0
u_dgr      UNCONN      0           0                      /var/run/chrony/chronyd.sock 19468                      * 0
u_str      LISTEN      0           5                             /mnt/wslg/PulseServer 19513                      * 0
u_str      LISTEN      0           100                   /mnt/wslg/PulseAudioRDPSource 20512                      * 0
u_str      LISTEN      0           100                     /mnt/wslg/PulseAudioRDPSink 24600                      * 0
u_str      LISTEN      0           4096                           /tmp/dbus-clzxK5Ywbg 18463                      * 0
```


# 3 比较 Loopback-Interface und Unix Domain Sockets


Lokale Services können statt Unix Domain Sockets auch Netzwerkverbindungen zu _localhost_ über das Loopback-Interface verwenden. Welche Vorteile haben Unix Domain Sockets?

Antwort
- Verbindungen sind etwas effizienter, weil der Overhead des TCP/IP-Stacks (Paketheader, Routing, Firewall) entfällt.
- Benutzerspezifische Pfade ermöglichen User-spezifische Services
- Automatische Authentifizierung über Dateizugriffsrechte

# 4 Unbenannte Sockets

Es gibt auch unbenannte (_unnamed_) Unix Domain Sockets, die nicht persistent im Dateisystem gespeichert werden. Sie werden in der Praxis seltener verwendet. Der System Call [socketpair()](https://man7.org/linux/man-pages/man2/socketpair.2.html) erzeugt ein Paar verbundener unbenannter Sockets, die dann z.B. an Kindprozesse weitergegeben werden können.

Linux kennt zusätzlich sogenannte **abstrakte Sockets**. Diese Sockets existieren außerhalb des Dateisystems und haben eindeutige Namen aus einem eigenen Namensraum.
