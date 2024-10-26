POSIX definiert die implementierungsabängige Logging-Schnittstelle `syslog()`. Logging-Nachrichten bestehen aus einem Text und einer Priorität. Log-Dateien werden im Verzeichnis `/var/log` abgelegt.

# 1 Log 文件位置 


Die Log-Dateien liegen im Verzeichnis `/var/log`. 
Häufige Dateinamen für das _system log_ sind `/var/log/syslog` und `/var/log/messages`


Ubuntu verwendet den bei systemd mitgelieferten Logger `systemd-journald`. Dieser legt die Logs im Verzeichnis `/var/log/journal` ab. Die Abfrage erfolgt mit dem Kommando `journalctl


# 2 Logging-Schnittstelle `syslog()`


Der POSIX-Standard definiert eine implementierungsunabhängige **Logging-Schnittstelle**: Mit der Bibliotheksfunktion `syslog()` können Prozesse Nachrichten in das _system log_ schreiben.



# 3 Kernel Messages ( Kommando `dmesg` )

Der Kernel speichert Kernel- und Hardware-bezogene Ereignisse aus Performance-Gründen nicht persistent, sondern in einem dafür reservierten Puffer begrenzter Kapazität im RAM


Die gespeicherten Meldungen kann man mit dem Kommando `dmesg` anzeigen


