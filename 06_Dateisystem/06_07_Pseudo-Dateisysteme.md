

In Unix gibt es für besondere Aufgaben **Pseudo-Dateisysteme** (_pseudo file systems_), die nicht physisch auf einem Datenträger existieren. Sie werden vom Kernel virtuell erzeugt und dienen zur temporären Datenspeicherung oder als Schnittstelle für Informationen und Parameter des Kernels.

Pseudo-Dateisysteme sind nicht standardisiert; gerade zwischen Linux und BSD-Varianten gibt es hier wesentliche Unterschiede. Der Rest dieses Abschnitts fokussiert auf wichtige Beispiele in Linux.

- Ein **tmpfs** (_temporary file system_) ist vom Kernel im RAM abgebildetes Dateisystem (_ramdisk_) zur Ablage temporärer Dateien. Es wird wie ein normales Dateisystem gemountet und verwendet. Zugriffe sind sehr schnell, weil alle Daten im RAM liegen. Der Inhalt geht beim Herunterfahren des Systems verloren.
    - Das von vielen Anwendungen genutzte Verzeichnis `/tmp` für temporäre Dateien wird gerne als _tmpfs_ realisiert. Dadurch wird die Performance erhöht und Verschleiß der Festplatte vermieden.
- `/dev` ist heute meistens ein spezielles **devtmpfs** im RAM, das vom Kernel beim Systemstart erzeugt und dynamisch mit den Gerätedateien gefüllt wird. Unter Linux ist dafür der Systemdienst `udev` zuständig.
    - Traditionell war `/dev` ein Verzeichnis des root-Filesystems mit statischen bei der Systeminstallation erzeugten Gerätedateien. Da aber heute Geräte oft zur Laufzeit angeschlossen und entfernt werden (_hotplug_, z.B. USB), ist ein dynamisches Pseudo-Dateisystem effizienter.
- **sysfs** (mount point `/sys`) ermöglicht den überwiegend lesenden Zugriff auf verschiedene Kernel-Datenstrukturen. Einige Kernel-Parameter können hier auch geschrieben werden.
    - Kernel 中数据的录取
- **proc** (mount point `/proc`) ist ein Pseudo-Dateisystem für **Prozess-Informationen**. Über die enthaltenen Dateien kann man — meistens nur lesend — auf Kernel-Informationen zu einzelnen Prozessen zugreifen. Mehr dazu erfahren Sie in [Lerneinheit 7](https://moodle.oncampus.de/modules/ir866/onmod/proc/process-info.html). Das Verzeichnis `/proc/sys` enthält globale Kernel-Parameter, die zum Teil auch (direkt oder mit dem Kommando `sysctl`) geschrieben werden können.
    - Die Struktur von _proc_ ist historisch gewachsen und wirkt nicht immer ganz logisch. Man findet hier auch globale System-Parameter, die besser in das neuere _sysfs_ passen würden, aber aus der Kompatibilität wegen in _proc_ geblieben sind.
    - Beispielsweise enthalten sowohl `/proc/sys/vm` als auch `/sys/kernel/mm` Informationen zum Speichermanagement.
