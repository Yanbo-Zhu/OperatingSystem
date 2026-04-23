
在 /dev/下  有 Gerätedateien
Der Zugriff von Unix-Anwendungen auf Hardware-Geräte erfolgt über spezielle Dateien, die **Gerätedateien (device files, device nodes)** im Verzeichnis `/dev/`.

Gerät hat zuständigen Gerätetreiber (device driver) 

Jedes Gerät im System wird durch zwei Nummern identifiziert, die man im Output von `ls -l` sieht:

1. Die **major device number** identifiziert den zuständigen Treiber
2. Die **minor device number** identifiziert ein konkretes von diesem Treiber kontrolliertes Gerät

```sh
# 1 就是 **major device number** ,   8 是**minor device number** 

$ cd /dev; ls -l rtc0 sd* tty?
crw------- 1 root root 248, 0 May 28 11:39 rtc0
brw-rw---- 1 root disk 8, 0 May 28 11:39 sda
brw-rw---- 1 root disk 8, 1 May 28 11:39 sda1
brw-rw---- 1 root disk 8, 2 May 28 11:39 sda2
crw--w---- 1 root tty  4, 0 May 28 11:39 tty0
crw--w---- 1 gdm  tty  4, 1 May 28 11:39 tty1
crw--w---- 1 root tty  4, 2 May 28 11:39 tty2
crw--w---- 1 root tty  4, 3 May 28 11:39 tty3
```


# 1 zwei Kategorien von Geräten:

Zeichenbasierte Geräte (**character devices**, Dateityp `c` im Output von `ls -l`) sind Geräte, bei denen der Datenaustausch Byte für Byte erfolgt. Meistens sind das langsamere Geräte mit relativ geringem Datenvolumen wie Terminal, Tastatur, Maus, Sound-Geräte und serielle Schnittstellen.

Blockgeräte (**block devices**, Dateityp `b`) sind schnelle Geräte für große Datenmengen, bei denen Daten in Blöcken fester Länge (typisch: 512-8192 Bytes) übertragen werden. Man muss also immer ganze Blöcke auf einmal lesen oder schreiben. Dazu gehören Datenträger wie Festplatten und SSDs.

Gerätedateien werden mit dem System Call `mknod()` oder dem Kommando `mknod` grundsätzlich unter jedem Pfad erzeugt werden; per Konvention gibt es sie aber normalerweise nur in `/dev`






# 2 Pseudo-Geräte


Es gibt **Pseudo-Geräte**, die keine Hardware repräsentieren, sondern spezielle Funktionen haben. Einige davon findet man auf praktisch allen Unix-Systemen, sie werden auch oft in Shell Scripts verwendet:

|                |                                                                                                         |
| -------------- | ------------------------------------------------------------------------------------------------------- |
| `/dev/null`    | "Schwarzes Loch": Geschriebene Daten werden verworfen, Lesen liefert immer EOF .   就是说放到这里面的文件, 会被自的删除  |
| `/dev/tty`     | Das dem aktuellen Prozess zugeordnete _controlling terminal_                                            |
| `/dev/urandom` | Pseudozufallsgenerator, liefert beliebig viele Pseudozufalls-Bytes                                      |
| `/dev/zero`    | Null-Generator, liefert beliebig viele Null-Bytes                                                       |




