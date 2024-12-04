
Der POSIX-Standard definiert ein portables Ausführungsmodell und API zur Nutzung von Threads. Diese **POSIX Threads** (kurz: **pthreads**) sind das unter Unix meistgenutzte Thread-Konzept.

POSIX definiert nur das API, nicht die Implementierung. Insbesondere wird nicht spezifiziert, ob es sich um User- oder Kernel-level Threads handelt.

Um pthreads in einem C-Programm zu nutzen, muss man
- den entsprechenden Header inkludieren: `#include <pthread.h>` und
- über das Compiler-Flag `-pthread` die pthread-Library einbinden.

# 1 API-Funktionen

Von den zahlreichen API-Funktionen stellen wir hier nur die wichtigsten kurz vor:
Verschaffen Sie sich anhand der _man pages_ einen Überblick über die Arbeitsweise der oben genannten Funktionen.

|   |   |
|---|---|
|**API-Funktion**|**Aufgabe**|
|`pthread_create()`|erzeugt einen neuen Thread und führt darin eine gegebene Funktion aus|
|`pthread_exit()`|beendet den aktuellen Thread (vgl. `exit()`)|
|`pthread_join()`|wartet auf Ende eines Threads und gibt dessen Ressourcen frei (vgl. `waitpid()`)|
|`pthread_detach()`|_detached_ Threads werden nach ihrer Beendigung ohne join sofort aufgeräumt|
|`pthread_self()`|fragt die eigene Thread-ID ab|
|`pthread_cancel()`|sendet eine ignorierbare Beendigungsanfrage an einen anderen Thread|
|`pthread_kill()`|sendet ein Signal an einen Thread|

# 2 Implementierung in Linux


**In Linux sind POSIX Threads als Kernel-level threads implementiert**. Threads werden in der globalen Prozesstabelle verwaltet.


Mit `ps -L` können Sie Informationen zu Threads anzeigen.

Jeder Thread hat eine eigene globale **Kernel-Thread-ID (TID)**, die zusammen mit den Prozess-IDs aus dem gleichen Nummernkreis vergeben wird. Die TID ist aber nur für den Kernel relevant. Sie ist unabhängig von der im pthread-API verwendeten und nur lokal im Prozess gültigen POSIX-Thread-ID.


Verwenden Sie in der Anwendungsprogrammierung nur die von `pthread_create()` gelieferten POSIX-Thread-IDs.


# 3 Beispiel 

Die Datei `threads.c` enthält ein Beispiel für POSIX-Threads:

```c
#include <stdio.h>
#include <pthread.h>

// all threads run this function
void *counter(void *arg) {
  char message = (char)((long)arg);

  printf("%c: I am thread %ld\n", message, pthread_self());

  for (int i=0; i<10; i++)
    printf("%c: %d\n", message, i);

  pthread_exit(NULL);  (3) // return NULL;  here does the same 
}


int main() {

  // allocate memory for thread IDs
  pthread_t thread1, thread2, thread3;   (1)

  // create 3 threads, each runs counter() with a different argument
  pthread_create(&thread1, NULL, counter, (void *)'A');   (2)
  pthread_create(&thread2, NULL, counter, (void *)'B');
  pthread_create(&thread3, NULL, counter, (void *)'C');

  // wait for all threads to finish
  pthread_join(thread1, NULL);   ()
  pthread_join(thread2, NULL);
  pthread_join(thread3, NULL);

  return 0;
}
```


(1)	Hier wird Speicherplatz für die Thread-IDs reserviert. pthread_t ist ein geeigneter ganzzahliger Typ, z.B. unsigned long.
(2)	pthread_create() erhält 4 Pointer: den Speicher für die Thread-ID, optionale Thread-Attribute (hier NULL), die auszuführende Startfunktion, und ein anwendungsspezifisches Argument dafür.
(3)	pthread_exit() beendet den Thread und gibt einen void-Pointer als Return-Wert zurück. Beim Rücksprung aus der Startfunktion wird es automatisch ausgeführt (analog zu exit()).
(4)	pthread_join wartet auf das Ende eines Threads (analog zu waitpid()).


----

解释 pthread_create(&thread1, NULL, counter, (void *)'A');  

1  den Speicher für die Thread-ID,
Viele der pthread-API-Funktionen erwarten Pointer auf Datenstrukturen, die der Aufrufer vorher allokieren muss, z.B. für die Thread-ID. 

2 die auszuführende Startfunktion
Die als _function pointer_ übergebene Startfunktion hat die gleiche Rolle wie `main()` bei einem Prozess: Sie ist der Startpunkt des Threads und beim Rücksprung daraus wird der Thread automatisch beendet.

Gewöhnungsbedürftig ist die Verwendung von `void`-Pointern als Argument und Rückgabewert in der Startfunktion: Dieser _unspezifische Pointer-Typ_ `void *` ermöglicht es, Zeiger auf beliebige Datentypen zu übergeben. Sie müssen aber im Code von/in `void *` umgewandelt werden.    
当 return (xx), xx 是一个任意的数据类型, function head 中的void * 都可以把他 转化为 zeiger 

Kurze Daten (bis zur Länge eines Pointers, auf 64-Bit-Architekturen typischerweise 64 Bit) können auch durch einen direkten _type cast_ übergeben werden, ohne tatsächlich einen Zeiger zu nutzen. Dies haben wir im Beispiel mit einem `char`-Argument gemacht, das als void-Pointer "getarnt" 伪装 übergeben wird.




