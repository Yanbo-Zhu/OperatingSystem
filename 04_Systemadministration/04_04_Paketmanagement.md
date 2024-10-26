

# 1 deb-Pakete


**deb-Pakete** sind das von Debian und Ubuntu verwendete binäre Paketformat. Ein deb-Paket ist ein Archiv im (ziemlich alten) `ar`-Format . 


# 2 apt 

Das Verzeichnis `/etc/apt` enthält ==Konfigurationsdateien für das Paketmanagement-System==.
In der Datei `/etc/apt/sources.list` sind die zu verwendenden ==Paketquellen== (Repositories) aufgeführt.

`apt-get` und `apt-cache`.

|   |   |
|---|---|
|`apt update`|Informationen über verfügbare Pakete downloaden|
|`apt upgrade`|Alle installierten Pakete aktualisieren (vorher immer `apt update` ausführen!)|
|`apt install`|Pakete installieren|
|`apt remove`|Pakete deinstallieren|
|`apt search`|Nach Paketen suchen|
|`apt depends`|Abhängigkeiten eines Pakets anzeigen|


