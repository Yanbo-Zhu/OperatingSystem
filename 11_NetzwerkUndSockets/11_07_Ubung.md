
# 1 Einfache Firewall mit nftables

Implementieren und testen Sie eine einfache Firewall mit nftables auf Ihrer Ubuntu-VM wie im [nftables Tutorial](https://u-labs.de/portal/einstiegt-in-nftables-so-funktioniert-die-neue-firewall-unterschiede-zu-iptables/) beschrieben. Verwenden Sie dazu das Kommando `nft` und erstellen Sie eine passende Konfigurationsdatei mit nftables-Regeln. Ihre Firewall soll
- beliebigen Traffic auf dem Loopback-Interface akzeptieren
- alle von der VM nach außen aufgebauten Verbindungen und die Antworten darauf akzeptieren
- von außen eingehende SSH-Verbindungen akzeptieren
- eingehende Pings und Antworten darauf zulassen

---

Kommentierte Konfigurationsdatei: `unix-code/solutions/11/nft-simple-firewall.conf`. Wichtige Kommandos (als root ausführen):
- Laden: `nft -f nft-simple-firewall.conf`
- Aktuelle Regeln ausgeben: `nft list ruleset`
- Alle Regeln löschen: `nft flush ruleset`

# 2 Time-Server über TCP

1. Schreiben Sie ein Programm `time-server.c`, das auf dem TCP-Port 54321 hört und in einer Endlosschleife
    - auf eine eingehende Verbindung wartet
    - IP-Adresse und Port des Clients auf stdout ausgibt
    - Datum und Uhrzeit an den Client sendet (Tipp: `time()`, `localtime()`, `strftime()`)
    - und dann die Verbindung schließt

Sie können Ihr Programm mit `nc` (_netcat_) als Client testen:
```
$ nc 127.0.0.1 54321
Sun Jun 26 20:23:48 2022
```


2. Modifizieren Sie Ihr Programm zu einer Variante `time-server-6.c`, die IPv6 nutzt (Tipp: `AF_INET6`, `struct sockaddr_in6`, `in6addr_any`, `inet_ntop()`).


---

Siehe `unix-code/solutions/11/time-server.c` und `time-server-6.c`.

# 3 UDP-Chat-App

Entwickeln Sie ein UDP-basiertes Programm `chat.c`, mit dem zwei Parteien bidirektional chatten können: `./chat <listen_port> <remote_ip> <remote_port>` empfängt eingehende Nachrichten auf `<listen_port>` und sendet ausgehende Nachrichten an `<remote_ip:remote:port>`.

Die Nachrichten bestehen jeweils aus einer Zeile Text. Arbeiten Sie mit zwei Threads, die denselben Socket verwenden: ein Thread sendet, der andere empfängt.

Zum lokalen Test starten Sie das Programm in zwei verschiedenen Terminals und verwenden 2 Ports von _localhost_ (127.0.0.1) spiegelbildlich:
1. `./chat 55550 127.0.0.1 55551`
2. `./chat 55551 127.0.0.1 55550`

---

Auch mit UDP genügt ein Socket als Endpunkt für beide Richtungen.
- Deklarieren Sie den Socket und die remote-Adresse als globale Variablen, damit sie von den Threads leicht genutzt werden können.
- Erzeugen Sie den Socket und binden ihn an den eingehenden Port `listen_port` und beliebige IP-Adresse (`INADDR_ANY`).
- Dann starten Sie den Sender- und Empfänger-Thread.
- Der Sender-Thread liest in einer Endlosschleife jeweils eine Zeile Text von stdin ein und sendet sie mit `sendto()` an die Gegenseite.
- Der Empfänger-Thread wartet in einer Endlosschleife jeweils mit `recv()` auf eine eingehende Nachricht und gibt sie auf stdout aus.
- Das Programm wird einfach mit Ctrl+c beendet. Dabei wird der Socket automatisch geschlossen. Weitere "Aufräumarbeiten" sind nicht erforderlich.

# 4 Logger mit Unix Domain Socket

Eine typische Anwendung von Unix Domain Sockets: Entwickeln Sie einen Logging-Server `logger.c`. Dieser ermöglicht es beliebigen Prozessen, Logging-Nachrichten zu schicken.
- Der Logger hört auf dem Unix Domain Socket `/tmp/logger.sock`, der für alle Benutzer des Systems zugreifbar ist. Der Socket verwendet das Protokoll `SOCK_STREAM`.
- Clients senden eine Nachricht, die aus einer Zeile Text (max. 128 Zeichen) besteht.
- Der Logger fragt die PID und UID des Clients ab (`getsockopt()` mit `SO_PEERCRED`).
- Empfangene Nachrichten werden im Format `PID (UID): Nachricht` auf stdout ausgegeben. (In einer echten Anwendung würde man noch einen Zeitstempel hinzufügen und die Nachrichten in einer Log-Datei speichern, aber das sparen wir uns hier.)

Mit dem Kommando `socat` (`sudo apt install socat`) können Sie den Logger testen:
`echo -n "Das ist eine Testnachricht" | socat - UNIX-CONNECT:/tmp/logger.sock`


---

- `bind()` schlägt fehl (_address already in use_), wenn die Socket-Datei schon existiert. Also muss sie vorher mit `unlink()` gelöscht werden. Lesen Sie [unix(7)](https://man7.org/linux/man-pages/man7/unix.7.html) zur Verwendung von Unix Domain Sockets.
- Auch beim Beenden des Loggers mit `Ctrl-c` sollte die Socket-Datei durch einen entsprechenden _signal handler_ gelöscht werden.
- Der Ablauf ist wie bei einem TCP-Socket:
    - Zur Initialisierung `socket()` - `bind()` - `listen()`
    - Dann in einer Endlosschleife `accept()` - `recv()` - `close()`.
- Nach `bind()` kann man den Socket mit `chmod()` für alle Benutzer schreibbar machen.
- Sie PID, UID und GID der Gegenseite erhält man nach `accept()` über die Socket-Option `SO_PEERCRED` (_peer credentials_). Damit die benötigte Datenstruktur `struct ucred` bekannt ist, müssen Sie `_GNU_SOURCE` definieren.

```c
#define _GNU_SOURCE
struct ucred peercred;
socklen_t peercred_len = sizeof(struct ucred);
err = getsockopt(sock, SOL_SOCKET, SO_PEERCRED, &peercred, &peercred_len);
```

# 5 File-Server und -Client mit TCP

答案: http://www.mario-konrad.ch/blog/programming/multithread/tutorial-04.html

_Optionale Aufgabe, etwas umfangreicher._

Implementieren Sie in C einen Client `file-client.c` und Server `file-server.c` für einen einfachen _read-only_ TCP-Fileservice nach folgender Spezifikation:

- Der Server wird in dem Verzeichnis gestartet, das die zu sendenden Dateien enthält. Einziges Argument ist die Portnummer, z.B. startet `file-server 44444` den Server auf Port 44444.
- Das Client-Programm hat 3 Argumente: IP-Adresse, Port, Dateiname. Der Dateiname ist optional.
    - `file-client 127.0.0.1 44444 hello.c` holt die Datei `hello.c` vom Server und schreibt sie nach stdout.
    - `file-client 127.0.0.1 44444` (also ohne Dateiname) listet die im Server-Verzeichnis verfügbaren Dateien auf.
- Der Server soll mehrere Client-Verbindungen gleichzeitig akzeptieren und behandeln können. Erzeugen Sie dazu für jede eingehende Verbindung einen neuen Server-Thread.
- Der Server öffnet alle Dateien _read-only_.
- Nur Dateien aus dem aktuellen Serververzeichnis dürfen zugreifbar sein. Der Server muss verhindern, dass Dateien aus anderen Verzeichnissen erreichbar sind.

Der Fileservice verwendet ein stark vereinfachtes _textbasiertes Anwendungsprotokoll_:
- Der Server meldet sich mit einer Textzeile, die mit "HELLO " beginnen muss (und ansonsten optionale Informationen enthalten kann, z.B. Serverversion und Status).
- Der Client prüft, ob die Begrüßungsmeldung tatsächlich mit "HELLO " beginnt. Wenn nicht, wird die Verbindung abgebrochen.
- Der Client schickt eine Textzeile mit einem Befehl. Es gibt nur zwei mögliche Befehle:
    - `DIR` — Liste die Namen der verfügbaren regulären Dateien aus dem aktellen Server-Verzeichnis auf, eine pro Zeile. Unterverzeichnisse und Symlinks werden ignoriert.
    - `GET filename` — Sende den Inhalt der benannten Datei, die eine reguläre Datei im Server-Verzeichnis sein muss.
- Der Server schickt die entsprechende Antwort und schließt die TCP-Verbindung.
- Der Client gibt die empfangene Antwort auf stdout aus und schließt die Verbindung. Um die empfangene Datei clientseitig abzuspeichern, muss der Client mit einer passenden Umlenkung aufgerufen werden.
- Zur Vereinfachung gibt es im Protokoll keine explizite Fehlerbehandlung. Bei einem Fehler (z.B. ungültiger Befehl, nicht vorhandene Datei) schließt der Server einfach die Verbindung, ohne Daten zu schicken.

