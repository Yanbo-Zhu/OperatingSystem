
ps 和 top 命令 
Informationen zu Prozessen lassen sich in Unix über verschiedene Wege abfragen.
- `ps` ist ein sehr flexibles Kommando zur Abfrage von Prozessinformationen als einmalige "Momentaufnahme".
- `top` und dessen modernere Variante `htop` geben eine interaktive, laufend aktualisierte Übersicht über die Prozesse und Auslastung des Systems.
- Mit den System Calls `getpid()` und `getppid()` kann ein Prozess seine eigene PID und die seines Elternprozesses abfragen; mehr dazu im [nächsten Abschnitt](https://moodle.oncampus.de/modules/ir866/onmod/proc/fork.html).
- Mit den System Calls `getuid()`, `geteuid()`, `getgid()`, `getgroups()` kann ein Prozess seine User- und Gruppenzugehörigkeiten abfragen.

1. Lesen Sie [man ps](https://man7.org/linux/man-pages/man1/ps.1.html) und testen Sie das Kommando mit den verschiedenen Optionen. Wie können Sie alle Prozesse im System, wie nur Ihre eigenen abfragen?
2. Installieren Sie `htop` (`sudo apt install htop`) und probieren Sie es aus. Welche Konfigurationsmöglichkeiten und unterschiedlichen Sichten gibt es?



`/proc` Filesystem
Das `/proc` Filesystem dient in Linux [[1](https://moodle.oncampus.de/modules/ir866/onmod/proc/process-info.html#_footnotedef_1 "View footnote.")] als Schnittstelle zu den internen Kernel-Datenstrukturen zu Prozessen. Im Unterverzeichnis `/proc/<pid>` finden Sie die Informationen zum Prozess `<pid>`. Erläuterungen dazu finden Sie in der [Kernel-Dokumentation](https://www.kernel.org/doc/html/latest/filesystems/proc.html)).

Sehen Sie das proc-Verzeichnis `/proc/$$` zu Ihrer aktuellen Shell an. Welche Informationen finden Sie dort? Welche sind öffentlich, welche nur für Ihren User zugänglich?

Unix-Systeme gehen mit Prozessinformationen relativ offen um:
- Jeder Benutzer kann die meisten Informationen aller Prozesse im System sehen, z.B. die Kommandozeile über `/proc/<pid>/cmdline`.
- Jeder Prozess kann über `/proc` das Environment aller anderen Prozesse desselben Benutzers über `/proc/<pid>/environ` lesen.


