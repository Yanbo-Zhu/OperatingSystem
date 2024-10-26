

- Ein Unix-System bootet in mehreren Phasen. Meistens startet die Firmware einen Boot Loader, der den Kernel lädt und den init-Prozess (PID 1) startet. Unter Linux wird dann zuerst ein _initramfs_ verwendet.
- Das Init-System konfiguriert und startet die für einen normalen Betrieb benötigten Systemkomponenten.
- - Moderne init-Systeme wie systemd berücksichtigen Abhängigkeiten zwischen Komponenten/Services und können lang laufende Service-Prozesse (daemons) überwachen und ggf. neu starten.

# 1 Boot-Vorgang
## 1.1 Boot用到的一些 东西 都放在 /boot 中 
Die Dateien für Boot Loader, Kernel und das (weiter unten erklärte) initramfs sind nach dem Booten im laufenden System üblicherweise im Verzeichnis `/boot` zu finden.

## 1.2 Boot-Vorgang
Boot-Vorgang besteht grob aus folgenden Schritten:

1. Nach dem Anschalten wird die CPU initialisiert und führt ein Programm im ROM- oder Flash-Speicher ("Firmware") ab einer "hart verdrahteten" Startadresse aus. Dieses Programm heißt **Boot-Firmware**.  
    1. initialisieren CPU 
    2. 执行**Boot-Firmware** in ROM
    3. Es gibt 2 PC-spezifische Ausprägungen der Boot-Firmware:
        - **BIOS** (_Basic I/O System_) ist älter und für einfachere Hardware gedacht.
        - **UEFI** (_Unified Extensible Firmware Interface_) ist moderner und flexibler erweiterbar.
2. Die Boot-Firmware initialisiert die Hardware, führt ggf. einen _power-on self test_ (POST) durch, sucht dann auf den vorhandenen Datenträgern (Platten etc.) einen **Boot Loader** und führt diesen aus.
    1. Boot-Firmware initialisiert die Hardware
    2. 执行 Boot Loader 
3. Der Boot Loader zeigt ggf. ein Boot-Menü an und startet dann den **Kernel** mit den ausgewählten Optionen.
    1. Boot loader 做了什么 
4. Der Kernel führt ein gegebenes Start-Programm als ersten Prozess im System aus. Dieser **init-Prozess** mit der Prozess-ID 1 startet dann schrittweise die weiteren Komponenten des Betriebssystems.
    1. Kernel 执行 Start-Programm 去创造一个 initial process 


## 1.3 Initramfs  (initialize file system)

启动文件系统

Das Booten von Linux findet meistens (aber nicht immer) zweistufig statt:
1. In der ersten Stufe startet der Kernel mit einem **initramfs** (_initial RAM file system_ ).   启动 Kernel mit initramfs 
    1. Dies ist ein komprimiertes Dateisystem-Archiv, das die für die erste Stufe notwendigen Dateien enthält:
        1. Treiber und Kernel-Module für Datenträger, Dateisysteme und ggf. Netzwerk
        2. Die in dieser Stufe verwendeten Dienstprogramme und Bibliotheken
        3. Ein geeignetes init-Programm, das als erster Prozess im System (PID 1) läuft.
2. Das initramfs wird vom Kernel entpackt und in den RAM geladen (daher der Name). Sein Inhalt wird in der ersten Stufe als _root file system_ verwendet, enthält also einen kompletten Dateibaum ab `/`.


Die Programme im initramfs dienen primär dazu, das vollständige _root file system_ verfügbar zu machen. Sobald dies erledigt ist, wird
- das initramfs durch das echte _root file system_ ersetzt, das normalerweise nicht mehr im RAM, sondern auf einem Datenträger liegt
- von dort ein neues init-Programm ausgeführt, das in der zweiten Stufe den weiteren Systemstart durchführt.


# 2 Init-System und Services

Booten der Kernel und der init-Prozess 之后 
就要开启 Init-System, 就是再去启动System-Komponenten

- Spezielle Hardware-Konfiguration, die nicht automatisch durch den Kernel erfolgt
- Einbinden aller benötigten Dateisysteme
- Konfiguration von Netzwerk-Schnittstellen und Routing
- Aktivierung von Login-Möglichkeiten, z.B. auf der Konsole oder über SSH
- Starten von Systemdiensten (Services), z.B. Logger, Web-Server, Datenbank-Server


## 2.1 rc-Scripts

 在建立Init-System的过程中,  `/etc/rc` 中的 shell scripts 会被执行 

Das traditionelle init-System in Unix besteht einfach aus einem Shell Script `/etc/rc`, das vom init-Prozess aufgerufen wird. Dieses Script startet nacheinander die entsprechenden Prozesse und ruft ggf. weitere Scripts (meist in `/etc/rc*` auf).

## 2.2 Zeitgesteuerte Jobs: cron und crontab


Dieser liest die globale Konfigurationsdatei `/etc/crontab` und führt die dort gelisteten Kommandos zu den angegebenen Zeitpunkten aus. 
Weiterhin hat jeder Benutzer eine eigene persönliche `crontab`-Datei, die mit dem Befehl `crontab` editiert werden kann
