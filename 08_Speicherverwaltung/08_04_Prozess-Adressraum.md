
==Der virtuelle Adressraum eines Prozesses== hat grob folgende Struktur 

![](images/Pasted%20image%2020241125223416.png)

Die einzelnen Speicherbereiche (_Segmente_ genannt) enthalten:

Text
    Der ausgeführte Programmcode.

Initialized data
    Globale/statische Variablen mit Initialwerten, die als Teil der Programmdatei hier abgelegt sind.

Uninitialized data
    Globale/statische Variablen ohne Initialwerte; dieses Segment wird zur Laufzeit mit 0 initialisiert.

Heap
    Bereich für die dynamische Allokation von Objekten zur Laufzeit, wächst in Richtung höherer Adressen.

Memory Mappings
    Mit `mmap()` abgebildete Dateien, z.B. dynamisch geladene _shared libraries_ (mehr dazu [später](https://moodle.oncampus.de/modules/ir866/onmod/mem/mmap.html#h_shared-libraries)).

Stack
    CPU-Stack mit Rücksprungadressen und lokalen Variablen, wächst in Richtung niedrigerer Adressen. Der **stack pointer**, ein CPU Register, zeigt immer auf das untere Ende des Stacks.

Die ersten drei Segemente werden zur Übersetzungszeit (genauer: vom Linker) festgelegt. Ihre Grenzen sind im C-Code als globale Symbole `etext, edata, end` zugänglich, siehe [end(3)](https://man7.org/linux/man-pages/man3/end.3.html) und das Codebeispiel darin.

In Linux liefern die Datei `/proc/<pid>/maps` und das tool `pmap` eine Übersicht über den virtuellen Adressraum eines Prozesses. Sehen Sie sich den virtuellen Adressraum der aktuellen Shell an:

`cat /proc/$$/maps` und `pmap $$`

Welche Segmente erkennen Sie?



# 1 Kernel Memory

Am oberen Ende des virtuellen Adressraums befindet sich der für den Kernel reservierte Speicher (_kernel memory_). 
Bei 64-Bit Architekturen ist hier unter anderem der gesamte physische RAM des Rechners auf einen zusammenhängenden virtuellen Adressbereich abgebildet.


所有 Prozesse 的 Kernel-Speicher 都有相同的 virtuellen Adressen. 这个 virtuellen Adressen 只有 cpu im _kernel mode_ 可以访问 
Der Kernel-Speicher ist _einheitlich für alle Prozesse auf die gleichen virtuellen Adressen_ abgebildet, aber nur im _kernel mode_ der CPU zugreifbar.

==Dies ist für eine effiziente Implementierung von System Calls notwendig: Weil der Kernel-Speicher immer an den gleichen virtuellen Adressen liegt, müssen die Seitentabellen beim Übergang in den _kernel mode_ nicht geändert werden.==


# 2 Heap

Der Heap enthält die von der Anwendung allokierten Objekte sowie Datenstrukturen zu deren Verwaltung.

## 2.1 Heap-Größe

Die Startadresse des Heaps  是随机分配的
Die Startadresse des Heaps wird auf vielen modernen Unix-Systemen beim Programmstart zufällig gewählt. Diese Sicherheitsmaßnahme erschwert bestimmte Arten von Buffer Overflow-Angriffen. Sie ist Teil der sogenanten **Address Space Layout Randomization** (ASLR).

 **program break** 标记的是 Die erste Adresse nach dem Heap =  das Ende des Heap
Die erste Adresse nach dem Heap nennt man **program break**; sie markiert das Ende des Heaps. Der Heap ist beim Start eines Prozesses recht klein (z.B. 100 kB) und wächst bei Bedarf in Richtung höherer Adressen.

System Call `brk()` oder der Variante `sbrk()` 可以改变Heap-Größe
Mit dem System Call `brk()` oder der Variante `sbrk()` fordert der Prozess beim Kernel eine Verschiebung des _program break_ und damit eine Änderung der Heap-Größe an.


Anwendungsprogramme sollten im Normalfall nicht direkt `brk()` aufrufen, sondern dies den API-Funktionen der Systembibliothek überlassen, die wir gleich vorstellen.

## 2.2 Verwaltung und Nutzung des Heaps

> die dynamische Allokation von Speicher für die Anwendung 

Der Kernel stellt nur den Heap-Speicher für den Prozess als Ganzes bereit. Die Verwaltung des Heap-Inhalts und die dynamische Allokation von Speicher für die Anwendung finden hingegen im _user space_ des Prozesses statt und sind in der C-Systembibliothek `libc` implementiert.

POSIX definiert dafür einige API-Funktionen (einige davon kennen Sie schon aus _C for Java Programmers_):
- `malloc()` allokiert einen zusammenhängenden Speicherbereich der angeforderten Größe auf dem Heap und liefert einen Pointer darauf. Der Speicher wird nicht initialisiert.
- `calloc()` allokiert Speicher für ein Array und initialisiert es mit 0.
- `free()` gibt einen vorher allokierten Speicherbereich (Pointer) frei.
- `realloc()` ändert nachträglich die Größe eines bereits allokierten Speicherbereichs; ggf. wird dieser dabei verschoben.

Die Implementierung dieser Funktionen pflegt interne Datenstrukturen zur Verwaltung des Heaps. Insbesondere muss gespeichert werden, welche Teile des Heaps aktuell belegt und welche frei sind.


Im Vergleich zur Seiten-basierten (je 4 kByte) virtuellen Speicherverwaltung des Betriebssystems muss d==ie Heap-Verwaltung viel feiner arbeiten==, da eine beliebige (auch kleine) Anzahl Bytes allokiert werden kann. Man benötigt dafür besonders optimierte Algorithmen und Datenstrukturen, um
- sehr schnell ein freies Speicherstück der passenden Größe zu finden und
- Fragmentierung des Heap-Speichers zu vermeiden

Die Heap-Größe wird automatisch an den Speicherbedarf angepasst, indem die API-Funktionen falls nötig den System Call `brk()` aufrufen.


## 2.3 Fehlerquellen

> Dynamische Speicherverwaltung ist eine häufige Fehlerquelle in Software: Das Anwendungsprogramm ist selbst dafür verantwortlich, dynamischen Speicher vor der Nutzung zu allokieren und nach der Nutzung wieder freizugeben. Fehler dabei können eine Anwendung auch angreifbar machen.

**Memory Management Unit (MMU)**, 

Die Hardware kann dabei nicht unterstützen: Aus Sicht des Kernels und der MMU sind alle Adressen im Heap gültig, da der gesamte Heap dem Prozess zugewiesen ist. Die MMU kann nicht prüfen, ob eine Adresse im Heap aktuell wirklich allokiert ist.

Typische Fehlerkategorien sind:

- _Heap-based buffer overflow_: Die Grenzen eines dynamisch allokierten Speicherbeichs werden unerkannt überschritten (z.B. bei einem Array, weil Arraygrenzen in C nicht geprüft werden). Dadurch können andere Objekte auf dem Heap gelesen oder geschrieben werden.
    
- _Use after free_: Ein dynamisches Datenobjekt wird mit `free()` freigeben. Irgendwo in der Anwendung (z.B. in einem Feld einer anderen Datenstruktur oder einer Variable in einem anderen Thread) gibt es aber noch einen Pointer darauf. Wenn dieser später noch einmal verwendet wird, ist der referenzierte Speicher vielleicht schon für ein anderes Objekt allokiert. Solche Fehler treten oft nur sporadisch auf.
    
- _Fehlende Initialisierung_: Aus Effizienzgründen initialisiert `malloc()` den allokierten Speicher nicht. Also können dort noch alte (ggf. vertrauliche) Daten aus einem früheren Objekt stehen.
    
- _Memory leaks_: in einem langlebigen Prozess wird immer wieder Speicher allokiert, aber nicht mehr vollständig freigegeben. Über die Zeit wächst der Heap immer weiter an und belegt unnötig Speicher.

# 3 Memory Mapping

> 将文件内容映射到 virtuellen Speicher eines Prozesses

**Memory Mapping** erlaubt es in Unix-Systemen, den Inhalt einer Datei direkt in den virtuellen Speicher eines Prozesses abzubilden. Dadurch wird der Zugriff sehr einfach und effizient. POSIX definiert dafür den System Call `mmap()`.


Die Datei `mmap.c` enthält ein sehr einfaches Beispiel:
```c
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/mman.h>

int main() {
  int fd = open("/tmp/mmap-test", O_RDWR | O_CREAT, 0644);
  ftruncate(fd, 16);  //  make file exactly 16 bytes long

  // map 16 bytes starting at position 0 into memory
  char* bytes = mmap(NULL, 16, PROT_READ|PROT_WRITE, MAP_SHARED, fd, 0);

  for(int i=0; i<16; i++)
    bytes[i] = 'A'+i;
}
```

in mmap() function , Mit `MAP_SHARED` wird bei Schreiboperationen im Speicher auch die unterliegenden Datei aktualisiert. Wann genau dies passiert, ist jedoch nicht festgelegt. Mit `msync()` kann ein Prozess das Herausschreiben von Änderungen in die Datei erzwingen.


was ändert sich, dass `MAP_SHARED` durch `MAP_PRIVAT` ersetzen
Die Datei ändert sich nicht. `MAP_PRIVATE` erzeugt ein privates _copy-on-write mapping_, d.h. Schreiboperationen wirken sich nur lokal im Prozess aus und haben keine Rückwirkung auf die Datei. Sie sind auch nicht für andere Prozesse sichtbar, die dieselbe Datei mappen.


Eine Datei kann auch gleichzeitig in mehrere Prozessen mit `mmap()` eingebunden werden. Schreibzugriffe müssen dann aber [synchronisiert](http://localhost:1313/modules/ir866/onmod/threads/sync.html) werden, um unerwünschte Effekte zu vermeiden.




## 3.1 Memory Mapping ohne Dateien

Mit `mmap()` kann man nicht nur Dateien in den Speicher abbilden, sondern auch
- I/O-Geräte abbilden (_memory-mapped I/O_)
- _Shared Memory_ nutzen, siehe [Lerneinheit 10](https://moodle.oncampus.de/modules/ir866/onmod/ipc/shm.html)
- Mit dem Flag `MAP_ANONYMOUS` (in BSD: `MAP_ANON`) wird ein virtueller Speicherbereich der angegebenen Größe allokiert und in den Adressraum abgebildet. `malloc()` nutzt dies zur Allokation großer Speicherblöcke.



## 3.2 Shared Libraries

Eine wichtige Anwendung von Memory Mapping sind **Shared Libraries**: Systembibliotheken wie `libc` werden in fast jedem Prozess benötigt. Um die ausführbaren Programme klein zu halten, ==werden diese Bibliotheken nicht statisch in jedes Programm eingebunden, sondern zur Laufzeit _dynamisch geladen_:==

Der compilierte Programmcode solcher Bibliotheken ist in _shared object_-Dateien (Endung `.so`) abgelegt. ==Zur Laufzeit bindet der **dynamic loader** beim Programmstart die benötigten _shared objects_ mit `mmap()` in den Adressraum des Prozesses ein und registriert die Adressen im Programmcode, damit die Library-Funktionen gefunden und aufgerufen werden können.==

Weil _shared object_-Dateien read-only sind, muss nur eine Kopie davon im physischen Speicher existieren, die in alle betreffenden Prozesse abgebildet wird. Dadurch ist dieser Mechanismus sehr speichereffizient.




