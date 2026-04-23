
In der `Kernel -Konfiguration` Wird festgelegt, 0b ein Feature fest im Kernel enthalten Oder als Modul nachladbar ist. 
Kernel-Module können manuell mit dem Befehl `modprobe` geladen Oder entfernt werden. Die beim Booten automatisch zu ladenden Module werden in` /etc/modules-load.d` definiert.




Wenn Hardware und Einsatzzweck eines Systems vorab genau kennt, kann man einen Kernel dafür spezifisch konfigurieren und bauen: Man aktiviert genau die benötigten Features und lässt alles andere weg.

So ein _customized kernel_ ist optimal auf das System angepasst und liefert gute Performance bei minimaler Größe. Andererseits ist er aber wenig flexibel: wenn sich etwas an der Hardware ändert oder z.B. ein anderer Dateisystemtyp verwendet werden soll, muss der Kernel neu konfiguriert und gebaut werden.

Im anderen Extrem könnte man einen "fetten" Kernel bauen, in dem sehr viele Features auf Vorrat aktiviert und die numerischen Optionen so eingestellt sind, dass sie für die meisten Einsatzgebiete passen. So ein Kernel ist universell einsetzbar, braucht aber deutlich mehr Plattenplatz und RAM und ist auch langsamer.

Dynamisch ladbare Kernel-Komponenten bieten einen Ausweg aus diesem Dilemma:
==Ein **Kernel-Modul** (_kernel module_) ist eine als separate Binärdatei verteilte Kernel-Komponente, die vom Kernel zur Laufzeit dynamisch geladen und wieder entfernt werden kann. Es stellt dem Kernel Schnittstellen in Form von exportierten Symbolen (Funktionen und globalen Variablen) zur Verfügung==.
Kernel-Modul 可以随时动态载入或者 去除, 这样 这个 kernel 就不fett了 


Ein Modul implementiert ein abgegrenztes Feature, z.B.
- den Treiber für eine spezielle Hardware
- einen zusätzlichen Dateisystemtyp
- einen Algorithmus zur Datenkompression oder Verschlüsselung

# 1 zwei Kategorien von Kernel-Modulen

Man unterscheidet zwei Kategorien von Kernel-Modulen nach Ihrer Quelle:
1. _in-tree modules_ gehören zum offiziellen _source tree_ des Kernels; sie werden vom jeweiligen Kernel-Projekt zentral gepflegt und offiziell unterstützt.
2. _out-of-tree modules_ werden aus separatem Quellcode gebaut. Sie sind nicht Bestandteil der Kernel-Distribution und werden nicht von dieser unterstützt.

_Out-of-tree_ Module werden vor allem verwendet für
- Spezielle Hardware-Treiber, die vom jeweiligen Hersteller entwickelt werden. Diese sind oft proprietär und werden nur in Binärform (also ohne Quellcode) verteilt. Typische Beispiele sind Treiber für bestimmte Grafikkarten oder Wifi-Interfaces.
- Neuere oder selten genutzte Open Source Komponenten. Eine Übernahme in den _source tree_ des jeweiligen Kernels erfolgt erst, wenn sie stabil sind und relativ große Nachfrage besteht.

Über Module kann man beliebigen Code in den Kernel einschleusen (渗入。溜入。走私进来). Daher sind das Laden und Entfernen von Modulen privilegierte Operationen, die root-Rechte (bzw. unter Linux die Capability `CAP_SYS_MODULE`) erfordern. Installieren Sie niemals Module aus nicht vertrauenswürdigen Quellen. Seien Sie besonders vorsichtig bei binären Modulen ohne verfügbaren Quellcode.

# 2 Kernel-Module in Linux

Wir gehen jetzt auf die Handhabung von Kernel-Modulen in Linux ein. Andere Unix-Systeme bieten abweichende, aber konzeptionell ähnliche Mechanismen.

**Ablage**: Kernel-Module sind in den meisten Distros in `/lib/modules/<kernel-version>` abgelegt [[1](https://moodle.oncampus.de/modules/ir866/onmod/kernel/modules.html#_footnotedef_1 "View footnote.")]. Dieses enthält verschiedene Dateien mit Metadaten der verfügbaren Module (`modules.*`). Die eigentlichen Module findet man unter `/lib/modules/<kernel-version>/kernel`.
```
yzh@BNB23350:~$ ll /lib/modules/
total 12
drwxr-xr-x  4 root root 4096 Dec 10 21:25 ./
drwxr-xr-x 67 root root 4096 Nov 20 21:34 ../
drwxr-xr-x  2 root root 4096 Nov 15 15:56 5.15.153.1-microsoft-standard-WSL2/
drwxr-xr-x  1 root root   40 Dec 20 14:36 5.15.167.4-microsoft-standard-WSL2/
```

**Dateiformat**: Module haben die Dateiendung `.ko` (_kernel object_). Es handelt sich um Binärdateien im ELF-Format, die die relevanten Symbole für ihre Schnittstellen exportieren.

**Laden/Entfernen**: Module werden mit dem System Call [init_module(2)](https://man7.org/linux/man-pages/man2/init_module.2.html) geladen und mit [delete_module(2)](https://man7.org/linux/man-pages/man2/delete_module.2.html) wieder entfernt. In der Shell kann man beides mit dem Kommando [modprobe(8)](https://man7.org/linux/man-pages/man8/modprobe.8.html) durchführen.

**Abhängigkeiten**: Ähnlich wie Softwarepakete können auch Module Abhängigkeiten zu anderen Modulen haben, weil sie deren Funktionen nutzen. Die Abhängigkeiten sind in der Metadaten-Datei `modules.dep` aufgelistet. Sie müssen beim Laden und Entfernen beachtet werden, um Inkonsistenzen zu vermeiden.

**Anzeigen**: Die virtuelle Datei `/proc/modules` enthält eine Liste der geladenen Module. [lsmod(8)](https://man7.org/linux/man-pages/man8/lsmod.8.html) zeigt diese in einem schöneren Format an.

**Automatisches Laden**: Viele Module (z.B. bekannte Hardwaretreiber oder Dateisysteme) werden vom Kernel bei Bedarf automatisch geladen, ohne dass man dies explizit veranlassen muss. Manche Module muss man aber explizit beim Booten laden. Sie werden in Konfigurationsdateien im Verzeichnis `/etc/modules-load.d` eingetragen, siehe [modules-load.d(5)](https://man7.org/linux/man-pages/man5/modules-load.d.5.html).

Module, die bereits im initramfs benötigt werden (z.B. um das root-Dateisystem nutzen zu können), müssen im initramfs eingepackt und geladen werden. Die Mechanismen dafür sind Distro-spezifisch, z.B. in Ubuntu über Konfigurationsdateien in `/etc/initramfs-tools`.

**Optionen und Blacklisting**: Manche Module haben konfigurierbare Optionen, die beim Laden gesetzt werden können. Diese kann man in Konfigurationsdateien im Verzeichnis `/etc/modprobe.d` hinterlegen, siehe [modprobe.d(5)](https://man7.org/linux/man-pages/man5/modprobe.d.5.html). Dort kann man auch explizit das automatische Laden bestimmter Module verhindern (_blacklisting_), wenn diese zu Konflikten oder Sicherheitsproblemen führen würden.


# 3 问答: 

1. Betrachten Sie den Inhalt von `/lib/modules/<kernel-version>/kernel`. Welche Kategorien von Modulen gibt es?
    
2. Welche Module sind aktuell geladen und welche Abhängigkeiten haben sie untereinander? → `lsmod` Finden Sie exemplarisch für einige davon heraus, wozu sie dienen.
    
3. Welche Module werden automatisch geladen? → `/etc/modules-load.d`
    
4. Welche Optionen werden gesetzt und welche Module sind _blacklisted_? → `/etc/modprobe.d` (Sie werden feststellen, dass für eine Virtuelle Maschine kaum etwas davon relevant ist.)
    
5. Laden Sie manuell das Modul `zfs` für das Dateisystem ZFS. Welche anderen Module werden aufgrund von Abhängigkeiten automatisch mit geladen?
    1. 5. Mit zfs werden automatisch als Abhängigkeiten die Module icp, spl, zavl, zcommon, zlua, znvpair, zunicode, zzstd installiert.
6. Entfernen Sie `zfs` wieder. Werden die vorher automatisch geladenen Abhängigkeiten ebenfalls entfernt?
    1. Die Abhängigkeiten werden nicht automatisch entfernt, da der Kernel nicht entscheiden kann, ob sie noch anderweitig gebraucht werden.


