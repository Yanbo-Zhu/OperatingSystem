
_Shared memory_ ist gemeinsam genutzter Speicher (RAM), der vom Betriebssystem verwaltet und in den virtuellen Adressraum der beteiligten Prozesse abgebildet wird.

Das **POSIX Shared Memory** API besteht aus den Funktionen `shm_open()` und `shm_unlink()`.

`shm_open()` öffnet ein _shared memory_-Objekt und gibt einen _file descriptor_ darauf zurück, über den es mit `mmap()` in den virtuellen Adressraum des aktuellen Prozesses abgebildet (built) werden kann. Die Größe eines neu angelegten Objekts muss zuerst mit dem System Call `ftruncate()` festgelegt werden, bevor man es `mmap`-en kann.

Ein _shared memory_-Objekt wird über einen mit `/` beginnenden Namen identifiziert und ist systemweit sichtbar. Es kann zum Lesen und/oder Schreiben geöffnet werden. Der Zugriff wird wie bei Dateien nach Benutzer- und Gruppen-ID über eine Zugriffsrechte-Maske (_mode_) kontrolliert. _Owner_ ist dabei der User, unter dessen ID das Objekt neu angelegt wurde.

_Shared memory_-Objekte sind persistent: sie existieren im Kernel, bis sie explizit gelöscht werden oder das System neu bootet. `shm_unlink()` löscht ein _shared memory_-Objekt und gibt den Speicher frei, sobald es von keinem Prozess mehr genutzt wird.

> Lesen Sie die _man pages_ [shm_overview(7)](https://man7.org/linux/man-pages/man7/shm_overview.7.html), [shm_open(3)](https://man7.org/linux/man-pages/man3/shm_open.3.html), [truncate(2)](https://man7.org/linux/man-pages/man2/truncate.2.html) und [mmap(2)](https://man7.org/linux/man-pages/man2/mmap.2.html).


---

Shared Memory ist sehr effizient: Da `mmap()` den geteilten Speicher über die Seitentabelle des Prozesses direkt in dessen virtuellen Adressraum integriert, ist der Zugriff darauf genauso schnell wie jeder andere Speicherzugriff. Der einzige "Overhead" besteht in der erforderlichen Synchronisation.

> POSIX Shared Memory beinhaltet keine Synchronisation. Der Zugriff darauf muss immer explizit synchronisiert werden, z.B. mit Semaphoren oder Mutexen.

困难: 
Eine praktische Herausforderung bei der Verwendung von _shared memory_ ist, dass alle beteiligten Prozesse den Namen des Objekts kennen müssen. Das ist schwierig, wenn es gleichzeitig mehrere Instanzen der betreffenden Programme geben kann, die mit separaten Objekten arbeiten sollen. Man muss dann den Namen entweder per Konvention festlegen oder zur Laufzeit über andere Mechanismen verteilen.



