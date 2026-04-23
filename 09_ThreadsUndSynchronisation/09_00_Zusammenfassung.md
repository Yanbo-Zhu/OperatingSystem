
- Threads sind Ausführungseinheiten, die innerhalb eines Prozesses nebenläufig verarbeitet werden und viele Ressourcen des Proesses (z.B. Speicher und offene Dateien) teilen.
- Man unterscheidet Kernel- und User-level Threads. In Multiprozessor-Systemen (z.B. Linux) verwendet man vorwiegend Kernel-level Threads, weil man damit mehrere CPUs nutzen kann.
- Der POSIX-Standard definiert ein Thread-API, die POSIX Threads (pthreads). Die Namen der zugehörigen API-Funktionen beginnen mit `pthread_`. Sie sind in der pthread-Library implementiert.
- 
- Wenn nebenläufig ausgeführte Prozesse oder Threads gemeinsame Ressourcen (z.B. globale Variablen) nutzen, kann es zu Race Conditions mit unerwünschtem Systemverhalten kommen.
- Zur Vermeidung von Race Conditions stellt das Betriebssystem Mechanismen zur Synchronisation bereit.
    
- Eine Sperre sichert einen kritischen Abschnitt ab. Das Blockieren und Freigeben einer Sperre muss atomar sein.
- Bei Sperren unterscheidet man zwischen Spinlocks 自旋锁 (mit aktivem Warten) und Mutexen 互斥 (mit Blockieren). Das pthread-API enthält Spinlocks und Mutexe für Threads.
- Semaphore 打旗语，发信号 信号量  sind eine Verallgemeinerung 普遍化 von Mutexen. POSIX definiert ein API für Semaphore; diese können _named_ (systemweit sichtbar) oder _unnamed_ (nur über _shared memory_ zugreifbar) sein.
    
- Der Zugriff auf Dateiabschnitte kann durch _advisory record locks_ synchronisiert werden.