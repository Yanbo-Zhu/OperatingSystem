

![](images/system-komponenten.svg)


# 1 Kernel mode and user mode 

## 1.1 Kernel mode 

Das privilegierten Modus (**kernel mode**)
- in dem sie Zugriff auf die gesamte Hardware, den kompletten physischen Speicher und alle Maschinenbefehle hat.

## 1.2 user mode

user mode 下的限制
In allen weiteren Komponenten arbeitet die CPU im unprivilegierten **user mode**, der wesentlichen Einschränkungen unterliegt:
- Nicht alle Maschinenbefehle können ausgeführt werden
- Kein direkter Zugriff auf die Hardware
- Statt dem physischen Speicher ist nur der _virtuelle Speicher_ des jeweiligen Prozesses zugreifbar

Für den Wechsel vom _user mode_ in den _kernel mode_ werden spezielle Maschinenbefehle verwendet, sogenannte Software-Interrupts (architekturspezifisch, z.B. TRAP, BRK, INT). Das System Call Interface ist daher nicht portabel.

其实会有多个不同的硬件权利层,   不只是 user mode and kernel mode 
Viele heutige CPUs haben mehr als 2 Hardware-Berechtigungsebenen (_Ringe_ genannt), die feiner abgestuft sind. In diesem Modul unterscheiden wir aber nur zwischen _user mode_ und _kernel mode_.

# 2 Kernel

Der **Kernel** (Betriebssystem-Kern) ist die hardwarenahe Ebene des Betriebssystems. Er verwaltet die Systemressourcen und stellt den Anwendungsprogrammen Dienste und Abstraktionen (z.B. Prozesse, Dateien, Geräte) bereit. Im Kernel läuft die CPU in einem privilegierten Modus (**kernel mode**), in dem sie Zugriff auf die gesamte Hardware, den kompletten physischen Speicher und alle Maschinenbefehle hat.
Kenerl 管理所有的 process, 文件, 设备 
Kernel mode 是区别于 user mode 的 
Kernel mode 下的 Kernel 对所有的硬件, Speicher 都有访问权 


# 3 System Call Interface

**Das System Call Interface  ist das API des Kernels**. Es besteht aus einer Anzahl definierter **System Calls** (kurz _syscalls_). Nur über diese können Anwendungsprogramme Dienste des Kernels aufrufen. Die System Calls sorgen für einen geordneten Wechsel vom unprivilegierten _user mode_ in den privilegierten _kernel mode_ und stellen die Einhaltung von Zugriffsbeschränkungen sicher. Die Funktionsweise von System Calls werden wir in [Lerneinheit 12](https://moodle.oncampus.de/modules/ir866/onmod/kernel/syscalls.html) genauer betrachten.

就是个API of kern, Anwendungsprogramme Dienste 通过这个去呼叫Kernel  
负责 从unprivileged user mode切换到  privileged kernel mode 


# 4 System library

System library 提供便捷的 Interfaces (C function), abtraction and services to the kernel. 
这个 functions  就叫做 system call. 
Application execute system call,   system call 通过 system call interface 传输到 kernel 去 接触 processer, hardware 等等 

**Systembibliotheken** (z.B. `libc, libpthread`) bilden eine Zwischenschicht zwischen den Anwendungen und den System Calls. 
Sie stellen auf der Ebene von C-Funktionen portable und zum Teil komfortablere Schnittstellen zum Kernel bereit und fügen zusätzliche Abstraktionen und Dienste hinzu (z.B. [POSIX Threads](https://moodle.oncampus.de/modules/ir866/onmod/threads/pthread.html), komfortablere Dateischnittstellen, feinere Speicherverwaltung). 
Die architekturspezifische Implementierung dieser Funktionen ruft dann die eigentlichen System Calls auf.


**System libraries** (e.g., `libc`, `libpthread`) form an intermediary layer between applications and system calls. They provide portable and, in some cases, more convenient interfaces to the kernel at the level of C functions and add additional abstractions and services (e.g., POSIX threads, more user-friendly file interfaces, finer memory management). The architecture-specific implementation of these functions then calls the actual system calls.

In der Praxis verwendet man bei der Systemprogrammierung die System Calls  (which are stored in System library ) fast nie direkt, sondern stattdessen die entsprechenden C-Bibliotheksfunktionen.

# 5 Shell and Standard-Utilities 

shell is a user interface, service program, 

Die **Shell** und **Standard-Utilities** stellen eine weitgehend standardisierte textbasierte Benutzeroberfläche sowie Dienstprogramme zur Systemadministration und Dateibearbeitung (z.B. `ls, ps, cp, rm, kill`) bereit. Mehr dazu erfahren Sie in den nächsten Lerneinheiten.


# 6 **Anwendungsprogramme** und **weitere Bibliotheken**

> no longer considered components of the operating system.
> 

Die eigentlichen **Anwendungsprogramme** und **weitere Bibliotheken** bauen auf den unteren Schichten auf. Sie werden im üblichen Sprachgebrauch nicht mehr als Komponenten des Betriebssystems betrachtet. Die Abgrenzung ist aber nicht eindeutig. Beispielsweise kann man einen C-Compiler als Anwendungsprogramm oder als Standard-Dienstprogramm ansehen.



