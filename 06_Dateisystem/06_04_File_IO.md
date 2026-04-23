
# 1 File Descriptors

Damit ein Prozess auf eine Datei zugreifen kann, muss sie erst mit `open()` geöffnet werden.

==Jede offene Datei wird im Prozess durch einen **file descriptor** repräsentiert. Dies ist eine nichtnegative, im Prozess eindeutige ganze Zahl.==

Die _file descriptors_ 0, 1, 2 sind immer automatisch vorhanden und mit _stdin, stdout, stderr_ verbunden.

Der System Call `dup()` dupliziert einen vorhandenen _file descriptor_: Es wird ein neuer _file descriptor_ erzeugt, der auf die gleiche Datei zeigt.

# 2 Ein Beispiel

Die Datei `read-write.c` enthält ein Beispiel zum Lesen und Schreiben von Dateien:

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#define BUFFER_SIZE 64

int main() {
  char buffer[BUFFER_SIZE];
  int nread, nwritten;

  int fd = open("./foo", O_RDONLY); 

  if (fd < 0) {
    perror("open foo failed");
    exit(1);
  }

  printf("opened fd %d\n", fd);

  nread = read(fd, buffer, BUFFER_SIZE); 
  fprintf(stderr, "%d bytes read\n", nread);

  if (nread > 0) {
    nwritten = write(1, buffer, nread);  
    fprintf(stderr, "%d bytes written\n", nwritten);
  }

  close(fd); 
}
```


1	open() öffnet die Datei foo zum Lesen (Flag O_RDONLY) und gibt einen file descriptor dafür zurück.
2	read() liest maximal BUFFER_SIZE Bytes aus der Datei in den bereitgestellten Puffer.
3	write() schreibt maximal nread Bytes aus dem Puffer nach stdout.
4	close() schließt die Datei foo wieder.

# 3 System Calls für I/O


Wir erläutern nun kurz die System Calls zur Datei-Ein-/Ausgabe.

- `open()` öffnet eine Datei und gibt im Erfolgsfall einen neuen _file descriptor_ zurück. Über verschiedene Flags wird unter anderem festgelegt,
    - ob die Datei zum Lesen, Schreiben oder für beides geöffnet wird
    - ob sie angelegt werden soll, wenn sie noch nicht existiert
    - ob geschriebene Daten sofort auf den Datenträger geschrieben werden sollen (_synchronized I/O_)
- `close()` schließt die referenzierte Datei und gibt den _file descriptor_ frei. Wenn der Prozess endet, werden offene Dateien automatisch geschlossen. Trotzdem ist es guter Stil, Dateien explizit zu schließen.
    
- `read()` liest maximal die angegebene Zahl von Bytes von der Datei in den bereitgestellten Puffer. Der Puffer muss von der Anwendung allokiert werden, seine Größe wird nicht geprüft. Rückgabewert ist die tatsächlich gelesene Anzahl von Bytes; der Wert 0 bedeutet _end of file_. Bei Fehlern wird -1 zurückgegeben.
    
- `write()` schreibt maximal die angegebene Zahl von Bytes von dem angegebenen Puffer in die Datei. Die Größe des Puffers wird nicht geprüft. Rückgabewert ist die tatsächlich geschriebene Anzahl von Bytes. Bei Fehlern wird -1 zurückgegeben.


# 4 Offset und seek


Lese- und Schreiboperationen beginnen grundsätzlich am Anfang der Datei und arbeiten sequenziell. Die aktuelle Position in der geöffneten Datei nennt man **file offset**, der Dateianfang hat Offset 0. Das Offset wird bei jedem `read()` und `write()` automatisch um die Anzahl der übertragenen Bytes erhöht.

Mit dem System Call `lseek()` kann man das Offset ändern, also innerhalb der Datei springen. Nicht alle Dateien sind _seekable_, Terminals beispielsweise nicht.







