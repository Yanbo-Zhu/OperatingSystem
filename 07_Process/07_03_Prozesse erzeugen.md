
# 1 Mit Fork()

In Unix-Systemen gibt es nur eine einzige Möglichkeit, einen neuen Prozess zu erzeugen: Ein laufender Prozess dupliziert sich selbst, indem er den System Call `fork()` aufruft.

`fork()` erzeugt eine _identische Kopie_ des aufrufenden Prozesses: 
Kindprozess 有新的PID和 Speicherbereich
Der neue **Kindprozess** (_child process_) hat eine neue PID und wird in einem separaten Speicherbereich ausgeführt. 

Kindprocess 和 Elternprozess 一样: Speicherinhalt 和 file descriptors 一样 
Er hat bei Rückkehr aus `fork()` initial den gleichen Speicherinhalt und wie ursprüngliche **Elternprozess** (_parent process_). Auch die _file descriptors_ des Elternprozesses werden kopiert.   

不一样的是 Speicher-Schreiboperationen und Zustandsänderungen
Danach erfolgen alle Speicher-Schreiboperationen und Zustandsänderungen in beiden Prozessen unabhängig voneinander.


---

Beide Prozesse setzen die Ausführung im Code direkt nach dem `fork()`-Aufruf fort. Der einzige Unterschied zwischen ihnen besteht im _Rückgabewert von_ `fork()`:
- Der Kindprozess erhält den Rückgabewert 0,
- der Elternprozess die PID des Kindes (oder -1 als Fehlercode, wenn kein Kind erzeugt werden konnte).

` child =fork()` 的返回值 = -1 证明 fork失败 , 等于0或者大于0 证明fork()成功了 

Durch Fallunterscheidung nach dem Rückgabewert kann erreichen, dass sich Eltern- und Kindprozess unterschiedlich verhalten — was normalerweise gewünscht ist, wie in diesem Beispiel:


```c
#include <sys/types.h> // for pid_t
#include <stdio.h>     // for printf(), perror()
#include <unistd.h>    // for fork(), getpid(), getppid()
#include <stdlib.h>    // for exit()

int main() {
  pid_t child;  // POSIX portable typedef, usually int.

  printf("parent %d started\n", getpid());
  child = fork();

  if(child < 0) {
    perror("fork failed");
    exit(EXIT_FAILURE);  // exit(1)
  } else if (child == 0) {
    printf("Child: %d with parent: %d\n", getpid(), getppid());
  } else {
    printf("Parent: created child with pid %d\n", child);
  }

  printf("hello\n");
  return 0;
}
```

# 2 Prozessbaum

`getppid()`,  pstree , htop, 

Beim Booten eines Unix-Systems wird als erster Prozess der [init-Prozess](https://moodle.oncampus.de/modules/ir866/onmod/admin/boot.html) mit der PID 1 gestartet. Jeder weitere Prozess im System wurde mit `fork()` erzeugt und hat genau einen Elternprozess. Dessen PID (_parent PID_) kann er mit dem System Call `getppid()` abfragen, siehe das Beispiel oben.

Alle Prozesse stammen daher direkt oder indirekt vom init-Prozess ab und bilden hinsichtlich der Eltern-Kind-Beziehung einen Baum mit dem init-Prozess als Wurzel.

Diesen **Prozessbaum** kann man mit dem Kommando `pstree` anzeigen. Das Tool `htop` liefert ebenfalls eine dynamische Baumstruktur der aktuellen Prozesse.

# 3 Prozessgruppen

Mehrere Prozesse eines Benutzers können zu einer logischen **Prozessgruppe (process group)** zusammengefasst werden. Dazu dienen die System Calls `setpgrp()` und `getpgrp()`. Die _Process Proup ID (PGID)_ ist die PID eines Prozesse aus der Gruppe, typischerweise des "führenden" Prozesses im Sinne der Anwendung. Bei `fork()` wird die PGID an den Kindprozess vererbt.

Prozessgruppen haben eine Bedeutung für das [Senden von Signalen](https://moodle.oncampus.de/modules/ir866/onmod/ipc/signals.html): Man kann mit `kill()` ein Signal an alle Prozesse einer Prozessgruppe schicken.

