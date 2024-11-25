

`fork()` erzeugt eine identische Kopie des aufrufenden Prozesses. Dazu müsste der Kernel eigentlich j==ede dem Elternprozess zugeordnete schreibbare Speicherseite (außer _shared memory_) kopieren und mit den Kopien eine neue Seitentabelle aufbauen.== Dieser aufwendige Kopiervorgang würde `fork()` bei speicherintensiven Prozessen stark verlangsamen.

Die Arbeit wäre auch vielen Fällen umsonst (徒劳的) : Oft führt der Kindprozess sofort mit `execve()` ein neues Programm aus. Dabei wird der Speicher sowieso komplett neu zugewiesen und die gerade eben mit viel Aufwand kopierten Seiten verworfen.


--- 
Das **copy-on-write** Verfahren: 
Zur Beschleunigung von `fork()` verwenden moderne Unix-Systeme das **copy-on-write** Verfahren:

Bei `fork()` wird nur die Seitentabelle kopiert, nicht die Speicherinhalte. Alle eigentlich beschreibbaren Seiten werden aber _im Eltern- und Kindprozess als read-only markiert_.

==Solange es auf eine Seite nur Lesezugriffe gibt, teilen sich beide Prozesse diese Seite. Erst wenn einer der beiden Prozesse auf die Seite schreibt, erzeugt die MMU einen Page Fault. Daraufhin kopiert der Kernel die Seite und ordnet die Kopie dem schreibenden Prozess zu.==

Dieses "faule" Kopieren ist sehr effizient, da nur die Seiten mit abweichendem Inhalt bei Bedarf kopiert werden.