
Threads eines Prozesses teilen sich dessen Speicher (insbes. die Variablen) und können darüber miteinander kommunizieren. Man muss die Zugriffe nur synchronisieren, um Race Conditions zu vermeiden.

Zwischen verschiedenen Prozessen ist es komplizierter, da diese getrennte Speicherbereiche haben. Hier benötigt man spezielle Mechanismen des Betriebssystems.

> **Interprozess-Kommunikation (IPC)** ist der Informationsaustausch zwischen mehreren Prozessen.

Man unterscheidet zwei prinzipielle Ansätze der IPC:

1. **Speicherbasierte IPC** beruht auf gemeinsamem Speicher, den sich die beteiligten Prozesse teilen. Dies ermöglicht einen sehr schnellen Datentransfer. POSIX definiert dafür einen _shared memory_-Mechanismus. Zugriffe darauf müssen normalerweise explizit synchronisiert werden, z.B. mit Mutexen oder Semaphoren.
2. Bei **nachrichtenbasierter IPC** senden/empfangen die beteiligten Prozesse Nachrichten über speziell dafür bereitgestellte Schnittstellen. Die Kommunikation kann auf verschiedene Arten erfolgen:
    1. unidirektional (nur in einer Richtung) oder bidirektional (beidseitig)
    2. synchron (mit Warten) oder asynchron (ohne Warten)

In Unix-Systemen gibt es verschiedene Ausprägungen nachrichtenbasierter IPC, die wir betrachten werden:
- Signale
- Pipes und FIFOs
- POSIX Message Queues
- Unix Domain Sockets ([nächste Lerneinheit](https://moodle.oncampus.de/modules/ir866/onmod/net/unix-sockets.html))

