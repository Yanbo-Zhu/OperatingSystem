
> **Swap Space** 的定义 : 本质是一个 space 
> Den für das Auslagern von Seiten (Paging) reservierten Speicherplatz auf einem persistenten Datenträger (meist Magnetplatte oder SSD) nennt man in Unix **Swap Space**.

virtuellen Speicherverwaltung 可以比 physischer RAM 获得更多的 Speicher.  不再需要的Page  可以交换到persistente Datenträger 
Ein wesentlicher Vorteil der virtuellen Speicherverwaltung ist, dass man den Prozessen im System insgesamt mehr Speicher zuweisen kann als physischer RAM vorhanden ist. Seiten, die gerade nicht benötigt werden, können auf persistente Datenträger ausgelagert werden (_Paging_).


如果 RAM 中找不到 page, 会报错, 然后将这个page从 persistente Datenträger  转移到 RAM 中 就可以捉取到他了. 
Wenn auf eine gerade nicht im RAM vorhandene Seite zugegriffen wird, generiert die MMU einen _Page Fault_, der zum Einlagern der betreffenden Seite in den RAM führt. Paging und verschiedene Seitenersetzungs-Algorithmen wurden im Module _Computerarchitektur und Betriebssysteme_ detailliert behandelt.


# 1 Anlegen und Aktivieren

In Unix gibt es zwei Möglichkeiten, Swap Space anzulegen:
1. Auf einem dafür exklusiv reservierten Datenträger, meist eine Partition der Systemplatte (**swap partition**)
2. In einer Datei fester Größe in einem regulären Dateisystem (**swap file**); dies wird nicht von allen Dateisystem-Typen unterstützt.

Swap Space wird mit dem Kommando `mkswap` erzeugt und mit `swapon` aktiviert. `swapon -s` zeigt den aktuell aktiven Swap Space an. Um Swap Space beim Booten automatisch zu aktivieren, tragen Sie ihn in die Dateisystem-Tabelle `/etc/fstab` ein.

Aus Performance-Gründen sollte für Swap Space ein relativ schneller Datenträger verwendet werden. Dies spricht für Flash-Speicher (SSD). Andererseits kommt es auf intensiv genutztem Swap Space zu vielen Schreiboperationen, was bei Flash-Speicher zu vorzeitigem Verschleiß führen kann.

> Swap Space ist um Größenordnungen langsamer als RAM. Er kann fehlenden RAM nicht ersetzen. Häufiges Ein- und Auslagern kann das System stark verlangsamen — bis hin zur Unbenutzbarkeit.



# 2 Wieviel Swap Space ist sinnvoll

Das hängt davon ab, wieviel RAM vorhanden ist und wie das System genutzt wird.

Eine bekannte pauschale Faustregel lautet "Swap = 0.5 * RAM". Dies passt aber oft nicht gut:
- Linux-Desktop-Systeme mit ≥ 8 GB RAM kommen im Normalbetrieb gut mit 1-2 GB Swap aus.
- Manche Einplatinen-Computer mit wenig RAM brauchen genauso viel Swap Space wie RAM,m überhaupt stabil zu laufen.
- Ein Cloud-Host mit vielen VMs, von denen immer nur ein Teil aktiv ist, kann sehr viel mehr Swap Space nutzen als er RAM hat, weil den VMs insgesamt viel mehr RAM zugewiesen wird als vorhanden ist (_overcommitment_).

> Am besten ermittelt man den Swap-Bedarf für ein konkretes System experimentell. Weil heute Plattenplatz billig ist, lohnt es sich aber in der Regel nicht, am Swap Space zu sparen.


Jedes Unix-System sollte Swap Space haben, auch wenn der vorhandene RAM nicht ausgenutzt wird. Lesen Sie dazu [diesen Artikel](https://linuxblog.io/linux-performance-almost-always-add-swap-space/). Hier ist auch beschrieben, wie man das Swap-Verhalten eines Linux-Systems durch Kernel-Parameter konfigurieren kann.


# 3 Speicherstatistik

Es gibt verschiedene Kommandos zur Auswertung der RAM- und Swap-Nutzung eines Systems:
- `free` zeigt den freien und belegten RAM und Swap Space an
- `vmstat` ermöglicht verschiedene statistische Auswertungen zur Systemaktivität; sehen Sie sich besonders `vmstat -s` an.
- Unter Linux enthält `/proc/meminfo` detaillierte Informationen zur Speichernutzung.
- `ps -v` zeigt den Speicherverbrauch je Prozess; durch _shared memory_, z.B. für Systembibliotheken, können die Zahlen aber ein verzerrtes Bild geben.
- `top` und `htop` zeigen ebenfalls die Speichernutzung je Prozess an

Probieren Sie diese Kommandos und ihre Optionen aus und lesen Sie die _man pages_ dazu.

# 4 Sicherheitsrisiken


ungültige Seiten im Swap Space 不会被删除, 以便之后的process 能够捉取到他.  但是这会造成 daten leak, 
Aus Performance-Gründen werden ungültige Seiten im Swap Space _nicht gelöscht_, sondern nur als frei markiert. Ein Angreifer mit Zugriff auf das Swap-Speichermedium kann daher auch später oft noch frühere Inhalte ausgelagerter Seiten lesen und so an vertrauliche Informationen kommen. Dadurch können auch Daten "leaken", die sonst auf einer verschlüsselten Platte gespeichert sind.

一些措施: 
Als Gegenmaßnahmen kann man
- den Swap Space verschlüsseln; dafür kann ein beim Booten zufällig generierter Schlüssel verwendet werden, dadurch wird der Inhalt beim Neustart unlesbar.
- den Inhalt beim Neustart immer komplett physisch löschen (_block discard_) - dies ist aber nur mit Flash-Speicher (SSD) praktikabel, auf Magnetplatten wäre es zu langsam.

# 5 Memory Locking

System Call 之后 ein Prozess 持续占用 ram 中的 Bereich.   这个Bereich 就不本办法 auslagern 到 swap space 上了 
Mit dem System Call `mlock()` kann ein Prozess einen angegebenen Bereich seines Adressraums im RAM resident halten, also eine Auslagerung der betreffenden Seiten verhindern. Dies ist solange gültig, bis die Sperre mit `munlock()` aufgehoben wird oder der Prozess terminiert. `mlockall()` und `munlockall()` sperren den gesamten RAM eines Prozesses bzw. heben die Sperre wieder auf.

Memory Locks können z.B. sinnvoll sein, um eine schnelle Reaktion des Prozesses auf externe Ereignisse zu ermöglichen. Sie werden bei `fork()` nicht vererbt.


