
# 1 Dateien Und Verzeichnisse

Alle Dateien eines Unix-Systems sind in eine globale hierarchische Struktur eingeordnet, den **Dateibaum**. Die Blätter dieses Baums sind die eigentlichen Dateien, die inneren Knoten sind **Verzeichnisse (directories)**. Die Wurzel des Baums ist das **root-Verzeichnis** `/`.


Im Gegensatz zu Windows hat Unix also keine Laufwerksbuchstaben; alle Dateien bilden einen einzigen Baum.

隐藏文件是以 . 开头的 
Versteckte Dateien/Verzeichnisse haben Namen, die mit einem Punkt . beginnen.   




# 2 Dateisysteme

Ein **Dateisystem (file system)** realisiert einen (Teil-)Dateibaum auf einem Datenträger, z.B. einer Partition einer Platte. 

Es speichert neben den Inhalten der Dateien/Verzeichnisse auch
- _Metadaten_ wie Zeitstempel, Eigentümer und Zugriffsrechte, Größe
- _Verwaltungsinformationen_, z.B. über freien/belegten Platz des Datenträgers
- _Indexstrukturen_ zum schnellen Zugriff auf Dateien

FAT32这个文件系统的Einschränkung
- maximal 4GB je Datei und 2TB je Datenträger
- Keine Dateizugriffsrechte und Eigentümer
- Anfällig gegen Datenkorruption, da kein _journaling_ verwendet wird


## 2.1 Bekannte Dateisystem-Typen

Einige bekannte Dateisystem-Typen für Unix sind:

|   |   |
|---|---|
|ext4|Älteres, stabiles und schnelles allround-FS für Linux|
|xfs|Moderneres Linux-FS besonders für sehr große Datenträger und Dateien|
|btrfs|Copy-on-write FS für Linux mit Subvolumes, Snapshots und integrierter RAID-Funktion|
|zfs|Copy-on-write FS mit leistungsfähigem Volume Manager, Snapshots und RAID-Funktion|
|ufs|Standard-FS für BSD-Systeme|
|FAT32|Plattformübergreifend, viele Einschränkungen, oft für USB-Speichermedien|

Auf die Eigenschaften und Konfigurations-Möglichkeiten der verschiedenen Dateisystem-Typen gehen wir hier nicht weiter ein.


## 2.2 Dateisysteme erzeugen

Ein Dateisystem wird erzeugt mit dem Kommando `mkfs.<typ> <Gerät>`.

`mkfs.ext4 /dev/sda1`  
erzeugt ein ext4-Dateisystem auf der Disk-Partition `/dev/sda1`

`mkfs` ist eine destruktive Operation, die ein schon vorhandenes Dateisystem ohne Nachfrage zerstört. Stellen Sie sicher, dass Sie Datenträger und Partitionsnummer korrekt angegeben haben!


`mkfs` hat je nach Dateisystem-Typ verschiedene Optionen zur Konfiguration des neuen Dateisystems, siehe die jeweilige Dokumentation.


Lesen Sie [mkfs.ext4(8)](https://linux.die.net/man/8/mkfs.ext4) zu den Konfigurations-Möglichkeiten eines ext4-Dateisystems.




# 3 Mount und Unmont 


## 3.1 Mount

Der System Call `mount()` und das gleichnamige Shell-Kommando `mount` hängen ein auf einem Blockgerät befindliches Dateisystem unter einem gegebenen Verzeichnis in den Dateibaum ein. Dieser **Mount Point** muss bereits als Directory im Dateibaum existieren. `mount` benötigt root-Rechte 


`mount -t ext4 /dev/sda1 /mnt`
hängt das auf der Partition `/dev/sda1` befindliche ext4-Dateisystem unter `/mnt` im Dateibaum ein.  将处于 `/dev/sda1` 分区上的 ext4-Dateisystem 挂在 /mnt 上面 


查看所有 system 中的 mounts 
Die Kommandos `mount` (ohne Argumente) oder `findmnt` geben eine Übersicht aller aktuellen _mounts_ im System aus.

## 3.2 Unmont 

Mit `umount()` bzw. `umount` (_unmount_) kann ein Dateisystem wieder ausgehängt werden. Auch dieses Kommando erfordert root-Rechte.



## 3.3 fstab


Konfigurationsdatei `/etc/fstab`     上面  定义的 dateisystem 都会被自动挂在 

Die im normalen Betrieb eines Systems verwendeten Dateisysteme sind in der Konfigurationsdatei `/etc/fstab` beschrieben. Sie enthält eine Liste dieser Dateisysteme mit den dazugehörigen mount points und mount-Optionen.

Beim Booten werden alle in `/etc/fstab` aufgelisteten Dateisysteme automatisch eingehängt, sofern dies nicht durch die Option `noauto` deaktiviert ist. Traditionell erfolgt dies über das Kommando `mount -a` in einem rc-Script.

Linux-Systeme mit systemd generieren beim Booten aus `/etc/fstab` für jedes Dateisystem eine _mount unit_ für systemd, die auch die Abhängigkeiten zu anderen Dateisystemen beschreibt. Dadurch kann auch das Mounten parallelisiert und Fehler besser behandelt werden.
Linux-Systeme  在重启的时候 会对每一个 Dateisystem  产生一个 mount unit fur systemd 






