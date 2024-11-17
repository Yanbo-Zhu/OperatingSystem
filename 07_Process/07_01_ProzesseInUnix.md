
Prozesse sind die grundlegende Einheit für die Ressourcenzuordnung im Betriebssystem

In Unix-Systemen verwaltet der Kernel die Prozesse über eine **Prozesstabelle**, die je Prozess einen **Prozesskontrollblock (PCB)** mit den dazugehörigen Informationen enthält.

In Unix-Systemen verwaltet der Kernel die Prozesse über eine **Prozesstabelle**, die je Prozess einen **Prozesskontrollblock (PCB)** mit den dazugehörigen Informationen enthält.

# 1 Prozesse_Zustande_Lebenszyklus


Die 5 wichtigsten Zustände mit Ihren im Output von `ps` sichtbaren Zustandscodes sind [[1](https://moodle.oncampus.de/modules/ir866/onmod/proc/processes.html#_footnotedef_1 "View footnote.")]:

- D — uninterruptible sleep (usually IO)
- R — running or runnable (on run queue)
- S — interruptible sleep (waiting for an event to complete)
- T — stopped by job control signal
- Z — defunct ("zombie") process, terminated but not reaped by its parent



In den _sleep_ Zuständen S und D ist ein Prozess blockiert und wartet auf ein externes Ereignis, z.B. Daten aus einem anderen Prozsess, die Beendigung einer I/O Operation oder eine Benutzereingabe. Wenn dieses Ereignis eintritt, wird der Prozess vom Kernel aufgeweckt (_wakeup_), also wieder in den Zustand _Runnable_ versetzt.

Prozesse im Zustand D (_uninterruptible sleep_) dürfen vor Eintreten des Ereignisses nicht unterbrochen werden. Typischerweise bedeutet das ein Warten auf eine I/O-Operation (D = _disk wait_), deren Abbruch zu einem inkonsistenten Systemzustand und zu Datenverlusten führen kann.

