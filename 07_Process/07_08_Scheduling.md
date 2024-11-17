
就是给 prozess 分配 cpu 的过程 
> Unter **Scheduling** versteht man die die Zuteilung von CPUs an ablaufbereite Prozesse oder Threads. Der **Scheduler** ist die dafür zuständige Kernel-Komponente.

Im Modul _Computerarchitektur und Betriebssystem_ wurden die Scheduling-Ziele diskutiert und Scheduling-Verfahren für unterschiedliche Einsatzzwecke vorgestellt. Lesen Sie bei Bedarf dort nach.

Für Unix-Systeme gibt es sehr verschiedene Scheduling-Verfahren. Diese können zum Teil in der Kernel-Konfiguration (zur Übersetzungszeit) ausgewählt und zur Laufzeit konfiguriert werden.

Der Scheduler ist nicht nur für Prozesse, sondern auch für sog. _Kernel-level Threads_ zuständig. In der Literatur zum Scheduling wird daher oft der allgemeinere Begriff "Thread" verwendet. Lassen Sie sich davon nicht verwirren.


# 1 Scheduling in Linux

In Linux ist jedem Prozess/Kernel-level-Thread eine Scheduling-Strategie (_scheduling policy_) zugeordnet. Es stehen verschiedene _scheduling policies_ zur Auswahl:

- `SCHED_OTHER`: für "normale" Prozess/Threads
    
- `SCHED_BATCH`: für Batch-Prozesse: weniger _preemption_, längere Zeitscheiben
    
- `SCHED_IDLE`: für sehr niederpriore Hintergrundjobs
    
- `SCHED_FIFO`, `SCHED_RR`, `SCHED_DEADLINE`: verschiedene Strategien für Prozesse mit Echtzeit-Anforderungen
    

Die Echtzeit-Strategien dürfen standardmäßig nur von privilegierten Prozessen verwendet werden. Scheduling-Strategie und -Parameter eines Prozesses können mit speziellen System Calls gelesen bzw. gesetzt werden, z.B. `sched_setscheduler(), sched_setparam()`.

Der **Completely Fair Scheduler** (CFS) ist der Default-Scheduler von Linux. Er ist für alle Threads zuständig, die keiner Echtzeit-Strategie unterliegen.==Ziel des CFS ist es, die CPU-Zeit gerecht/fair auf alle Threads zu verteilen, ohne blockierte/schlafende Threads zu benachteiligen.==

