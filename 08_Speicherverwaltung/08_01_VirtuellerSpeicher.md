
每个 process 都有自己的虚拟地址空间 
虚拟addressraum 被分为 page, 每页 4096 Byte

Moderne Unix-Systeme benutzen **virtuellen Speicher**: Jeder Prozess hat einen eigenen virtuellen Adressraum. Dieser ist logisch in _Seiten (pages)_ unterteilt.

Eine typische Seitengröße ist heute 4096 Byte. Die Seitengröße kann mit der POSIX-Funktion `sysconf()` abgefragt werden:

```
#include <unistd.h>
long pagesize = sysconf(_SC_PAGESIZE);
```

虚拟存储空间到物理存储空间 通过 MMU 映射 
每一个process 都应自己的一个 page table ,  
Seitentabellen 被 Kernel 管理

# 1 Kontextwechsel

Speicherzugriffe werden von der **Memory Management Unit (MMU)**, einer Hardware-Komponente, auf den physischen Speicher abgebildet wird. Dazu gibt es für jeden Prozess eine **Seitentabelle (_page table_)**, die aus Effizienzgründen mehrstufig aufgebaut ist. Seitentabellen werden vom Kernel verwaltet.

PCB: **Process Control Block** (Prozesssteuerblock)

Bei jedem Kontextwechsel muss der Kernel
- die Seitentabelle der bisherigen Prozesses in dessen PCB speichern
- die Seitentabelle des nächsten Prozesses aus dessen PCB in die MMU laden

# 2 CPU-Cache

Der **CPU-Cache** ist sehr schneller, in die CPU integrierter RAM, der als Zwischenspeicher dient und mehrfache Zugriffe auf dieselben Speicherzellen autmatisch beschleunigt. Der Cache arbeitet _transparent_, d.h. ein Programm kann einfach eine Speicherzelle adressieren. Er ist oft mehrstufig aufgebaut (L1-, L2-, L3-Cache) und wird von der Hardware verwaltet.

Auf manchen Architekturen muss das Betriebssystem bei Änderungen der Seitentabelle (z.B. einem Kontextwechsel) mit entsprechenden CPU-Instruktionen (指令) dafür sorgen, dass modifizierter Cache-Inhalt in den RAM zurückgeschrieben (_flush_) und der Cache dann als ungültig markiert (_invalidate_) wird.



