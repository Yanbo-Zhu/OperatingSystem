

Pipes eignen sich sehr gut zur Übertragung kontinuierlicher Datenströme zwischen Prozessen. 
Wenn  man hingegen Nachrichten variabler Länge übertragen will, die jeweils als Ganzes empfangen und verarbeitet werden müssen, sind Pipes unpraktisch. Für solche Fälle sind Message Queues besser geeignet.

> Eine POSIX Message Queue ist eine gepufferte Warteschlange zum Austausch von Nachrichten variabler Länge mit integrierter Synchronisation.

Eine POSIX Message Queue wird mit `mq_open()` erzeugt und geöffnet. Sie hat einen systemweit eindeutigen Namen, der auf `/` beginnt (wie Semaphore oder Shared Memory-Objekte). Der Zugriff wird wie bei Dateien über eine Zugriffsrechte-Maske nach User- und Group-ID gesteuert.

Die Message Queue stellt einen Puffer für eine maximale Anzahl (Linux-Default: 10) von Nachrichten begrenzter Länge (Linux-Default: je 8192 Byte) bereit.

Es handelt sich um eine _prioritätsgesteuerte Queue_. Nachrichten werden
- mit `mq_send()` gesendet, dabei kann eine Priorität angegeben werden.
- mit `mq_receive()` empfangen, dabei wird die die älteste Nachricht mit der höchsten Priorität geliefert.

Senden und Empfangen sind wie bei Pipes synchronisiert und blockieren, wenn die Queue voll bzw. leer ist


Lesen Sie [mq_overview(7)](https://man7.org/linux/man-pages/man7/mq_overview.7.html).

