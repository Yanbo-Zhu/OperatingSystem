
Die meisten Unix-Systeme haben einen **monolithischen Kernel**: ==Der Kernel ist ein zusammenhängendes Programm in einer großen (meist komprimierten) Binärdatei, die alle Kernel-Funktionen enthält.==

# 1 查 kernel version 
Beim Systemstart wird der Kernel vom Boot Loader als Ganzes geladen und ausgeführt. Die aktuell laufende Kernel-Version können Sie mit dem Kommando `uname -a` anzeigen.
```
yzh@BNB23350:~$ uname -a
Linux BNB23350 5.15.167.4-microsoft-standard-WSL2 #1 SMP Tue Nov 5 00:21:55 UTC 2024 x86_64 x86_64 x86_64 GNU/L
```


Bei Ubuntu finden Sie den Kernel in der Datei `/boot/vmlinux-<version>`,  
z.B. `/boot/vmlinuz-5.15.0-40-generic`. Die Kernel-Datei ist mit `bzip` komprimiert und in der genannten Version ca. 11 MB groß. Der entpackte Kernel belegt ca. 54 MB.


# 2 自己造 Kernel 和 kernel-konfiguration 

Der Quellcode eines Kernels wird in der Regel vom jeweiligen Projekt in einem zentralen Repository (meist [Git](https://git-scm.com/)) gepflegt. Der _source tree_ ist so vorbereitet, dass der Kernel mit `make` oder ähnlichen Tools automatisch gebaut werden kann.

Es wäre wenig sinnvoll, in jeden Kernel _alle_ verfügbaren Funktionalitäten, Dateisystemtypen, Hardwaretreiber etc. einzubauen — der resultierende Kernel wäre sehr groß und langsam. Manche Features schließen sich auch gegenseitig aus. Weiterhin gibt es statische Parameter, die vor dem Bauen definiert werden müssen und nicht zur Laufzeit veränderbar sind, z.B. die maximale Größe einer Kernel-Datenstruktur.


---

Daher muss der Kernel vor dem Bauen für die jeweilige Hardware und den Einsatzzweck **konfiguriert** werden: Über Konfigurationsdateien werden Features ausgewählt und Parameter gesetzt. Beim Bauen werden die Konfigurationsdateien durch entsprechende Tools/Scripts gelesen berücksichtigt.

Es gibt sehr unterschiedliche Strategien zur Kernel-Konfiguration, beispielsweise:
- Die von Distibutionen binär ausgelieferten _Desktop- und Server-Kernels_ sollen universell einsetzbar sein. Sie unterstützen die meiste gängige Hardware und Dateisysteme und sind entsprechend "fett".
- Kernel für _Cloud-Instanzen_ unterstützen nur virtualisierte Hardware und müssen schnell booten.
- Kernel für _embedded systems_ werden speziell für eine Hardware und einen Einsatzzweck gebaut. Sie sind auf Performance und geringe Größe optimiert; unnötige Komponenten lässt man weg.


# 3 Kernel-Konfiguration in Linux

Der Linux-Kernel wird über eine zentrale Konfigurationsdatei konfiguriert. Die Einträge haben die Form `CONFIG_name=value`. Es gibt über 9500 Konfigurationsvariablen (Ubuntu, Stand Juli 2022).

Die meisten davon sind "Schalter", die 2 oder 3 Werte annehmen können:
- `y`: yes, Feature aktivieren
- `n`: no, Feature nicht aktivieren
- `m`: module, Feature kann zur Laufzeit als [Kernel-Modul](https://moodle.oncampus.de/modules/ir866/onmod/kernel/modules.html) geladen werden (nur für als Modul verfügbare Features)

Daneben gibt es relativ wenige numerische oder String-Parameter. Für alle Einstellungen sind default-Werte definiert. Nur davon abweichende Werte muss man explizit konfigurieren.

---

Konfiguration 文件通过 `make menuconfig` 产生
Die Kernel-Konfiguration wird vorzugsweise über ein textuelles Menü editiert, das mit `make menuconfig` in der Wurzel des Quellcode-Baums aufgerufen wird. Dort sind alle Konfigurationseinstellungen und kurze Beschreibungen dazu über eine hierarchische Navigation erreichbar. Die erstellte Konfiguration wird in der Datei `.config` gespeichert, die dann beim Bauen des Kernels verwendet wird.

[The Linux kernel user’s and administrator’s guide](https://www.kernel.org/doc/html/latest/admin-guide/README.html) enthält allgemeine Hinweise zur Konfiguration und zum Bauen des Linux-Kernels.

---

Konfiguration文件放在了哪里 
In vielen Linux-Systemen ist die Konfiguration des aktuellen Kernels zur Laufzeit lesbar, z.B.
- In Ubuntu in der Datei `/boot/config-<name>`
- In anderen Distributionen oft in `/proc/config.gz`; diese Datei ist komprimiert und kann mit dem Tool `zcat` gelesen werden.

Sehen Sie sich die Kernel-Konfiguration Ihres Ubuntu-Systems an. Recherchieren Sie exemplarisch für einige der Einstellungen, was sie bedeuten.


# 4 Ubung: Einen minimalen Linux-Kernel bauen

- Geben Sie Ihrer Ubuntu-VM für diese Übung die maximal verfügbare Zahl an virtuellen CPUs (in der Regel = der Anzahl der physischen CPUs/Cores) sowie mindestens 2 GB RAM, besser 4 GB. Das müssen Sie im ausgeschalteten Zustand konfigurieren.
- Installieren Sie zuerst die nötigen Pakete:
    - `sudo apt install flex bison pkg-config libncurses-dev qemu-system-x86`
- Kopieren Sie den Kernel-Quellcode:
    - `git clone --depth 1 git://git.kernel.org/pub/scm/linux/kernel/git/gregkh/staging.git`

Die Option `--depth 1` ist dabei wichtig: Sie holt nur den neuesten Stand, der schon ca. 1,3 GB Plattenplatz belegt. Ohne diese Option wird die _gesamte git-Versionshistorie_ des Kernels (932.701 Commits) mit geladen. Das dauert lange (2,4 GB Download) und belegt ca. 3,8 GB Plattenplatz [[1](https://moodle.oncampus.de/modules/ir866/onmod/kernel/exercises.html#_footnotedef_1 "View footnote.")].

- Folgen Sie dann der Anleitung [Building a tiny Linux kernel](https://weeraman.com/building-a-tiny-linux-kernel). Den ersten Schritt (`git clone`) haben Sie eben schon gemacht. In folgenden Punkten müssen Sie von der Anleitung abweichen:
    - Die Anleitung verwendet zum Bauen die Option `make -j16`, die 16 parallele Jobs erzeugt; das sind wahrscheinlich zu viele. Verwenden Sie statt 16 die Anzahl der virtuellen CPUs Ihrer VM.
    - Das Tool zum Bauen des initramfs heißt jetzt `mk-initrd`.
    - Ersetzen Sie generell (auch im Script `mk-initrd`) den Befehl  
        `git clone [git@github.com](mailto:git@github.com):..` durch `git clone [https://github.com/](https://github.com/)..` . Sonst benötigen Sie ein Github-Konto mit hinterlegtem SSH-Key.


