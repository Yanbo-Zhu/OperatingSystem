

# 1 POSIX

==POSIX definiert nur Schnittstellen und Verhalten des Systems==; es macht keine Vorgaben zur Implementierung und ist hardwareunabhängig.
- _Base Definitions_: Definiert grundlegende Begriffe, Konzepte und Header-Dateien.
- _System Interfaces_: Definiert Schnittstellen des Systems (API, _application programming interface_) auf der Ebene von C-Funktionen, die in Standard-Systembibliotheken wie `glibc` verfügbar sein sollen.
- _Shell and Utilities_: Spezifiziert die Syntax und Funktionalität der Shell sowie der wichtigsten System-Utilities (z.B. `cat, cp, ls, mv, ps, rm`).


Die meisten heutigen Unix-Systeme sind weitgehend (aber nicht unbedingt zu 100%) **POSIX-konform**.


# 2 其他

- Single Unix Specification
    - Grundlage der [UNIX-Zertifizierungen](https://moodle.oncampus.de/modules/ir866/onmod/unix/unix-name.html)
- [Filesystem Hierarchy Standard (FHS)](https://refspecs.linuxfoundation.org/fhs.shtml)
    - definiert Konventionen für die Verzeichnisstruktur des Dateisystems von Linux-Systemen
- [Linux Standard Base (LSB)](https://refspecs.linuxfoundation.org/lsb.shtml)
    - binäre Schnittstelle für Anwendungen (ABI, _application binary interface_) auch manchen gängigen Prozessorarchitekturen. Ziel ist die Portabilität compilierter Applikationen in Binärformat.


  





