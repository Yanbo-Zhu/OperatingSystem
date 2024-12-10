
Pipes kennen Sie schon aus der Shell. Dahinter steckt ein allgemeiner IPC-Mechanismus für Unix-Systeme.

> Eine Pipe ist ein unidirektionaler 单向的。直流的。 gepufferter Bytestrom zwischen zwei Prozessen mit integrierter Synchronisation.

---


Eine Pipe wird mit dem System Call `pipe() `erzeugt. Dieser gibt ein Array von 2 file descriptors zurück: der erste für das Lese-Ende, der zweite für das Schreib-Ende der Pipe. Darüber kann wie auf eine normale geöffnete Datei mit `read()` und `write()` zugegriffen werden.

Pipes werden bei `fork()` an die Kinder des erzeugenden Prozesses weitergeben. Sie können also vom erzeugenden Prozess und dessen Nachkommen genutzt werden. Um über eine Pipe zu kommunizieren, benötigen die beteiligten Prozesse also einen gemeinsamen Vorfahren, der die Pipe erzeugt. 

Eine Pipe arbeitet nach dem FIFO-Prinzip (First In First Out): Die Daten kommen genau in der Reihenfolge, in der sie in die Pipe geschrieben werden, beim Lesen wieder heraus.

Ein Prozess darf immer nur auf ein Ende einer Pipe zugreifen. Um Fehler zu vermeiden, sollte das unbenutzte Ende vor dem ersten Zugriff auf die Pipe geschlossen werden.

对同一个 文件 可以同时开启多个 writer 和 reader 

![](images/Pasted%20image%2020241210004019.png)

# 1 Puffer und Synchronisation

Der in die Pipe integrierte **Puffer** dient als Zwischenspeicher für geschriebene Daten, bis diese gelesen werden. Der Puffer hat eine feste Größe. Unter Linux beträgt diese standardmäßig 64 kByte, kann aber nach Erzeugen mit `fcntl()` geändert werden.

Auf das Schreib- und Leseende einer Pipe können jeweils mehrere Prozesse zugreifen. Lese- und Schreibzugriffe werden **automatisch synchronisiert**:
- `write()` bei vollem Puffer blockiert, bis (durch Leseoperationen) wieder genügend Platz frei ist.
- `read()` bei leerem Puffer blockiert, bis genügend Daten in die Pipe geschrieben werden.
- `write()` ist zu einer bestimmten Datenmenge (Konstante `BUF_SIZE`, in Linux 4096 Bytes) _atomar_. Dadurch wird verhindert, dass Daten aus verschiedenen Schreibprozessen durchmischt werden.

Mit `fcntl()` kann man eine Pipe in den (seltener verwendeten) **non-blocking** Modus versetzen: Dann blockieren Lese- und Schreiboperationen nicht, sondern geben eine Fehlermeldung zurück.

# 2 Schließen der Pipe



Wenn alle file descriptors zum Schreib-Ende einer Pipe geschlossen sind, gibt jede weitere Leseoperation end-of-file zurück. Wenn alle file descriptors zum Lese-Ende einer Pipe geschlossen sind, wird bei einer Schreiboperation das Signal SIGPIPE an den schreibenden Prozess geschickt.

Sobald alle file descriptors an beiden Enden der Pipe geschlossen sind (also spätestens wenn alle beteiligten Prozesse beendet sind), wird die Pipe vom Kernel aufgeräumt und der Puffer freigegeben.

Lesen Sie [pipe(7)](https://man7.org/linux/man-pages/man7/pipe.7.html) und [pipe(2)](https://man7.org/linux/man-pages/man2/pipe.2.html).


# 3 Ein kleines Beispiel

Hier ist ein minimales Beispiel zur Verwendung einer Pipe (der Einfachheit halber ohne Fehlerbehandlung):

simple-pipe.c

```c
#include <stdio.h>
#include <unistd.h>

int main() {
  int mypipe[2];

  pipe(mypipe); // pipe musts be created before forking

  if (fork() == 0) {
    close(mypipe[1]); // child reads, close write end
    char buffer[16];
    printf("waiting for data to appear\n");
    read(mypipe[0], buffer, 16);
    printf("read from pipe: %s\n", buffer);
  } else {
    close(mypipe[0]); // parent writes, close read end
    sleep(3);
    char message[] = "Hello World!";
    write(mypipe[1], message, 13);
  };
}
```

# 4 Ubung 


Schreiben Sie ein Programm pipe.c, das zwei Programme in jeweils einem Kindprozess startet und über eine Pipe den Output des ersten Programms mit dem Input des zweiten verknüpft. Zu beiden Programmen können Argumente angegeben werden, die Trennung erfolgt mit '|' (in Anführungszeichen, damit die Shell das nicht als Pipe interpretiert). Beispiel: `./pipe ls -la '|' head -n 5 `hat den gleichen Effekt wie in der Shell` ls -la | head -n 5`.



解题思路
Der Elternprozess sucht die Position des | in den Argumenten und ersetzt das | durch einen NULL-Pointer, damit der erste Teil von argv später für execvp() verwendet werden kann. Dann erzeugt er eine Pipe und zwei Kinder, schließt beide Enden der Pipe, und wartet auf beide Kinder.

Jedes Kind schließt ein Ende der Pipe, verbindet das andere über dup2() mit stdin bzw. stdout, und führt mit execve() das dazugehörige Kommando aus. Codeausschnitt für eines der Kinder:

```c
close(fd[0]);
dup2(fd[1],1);
execvp(...);
```


## 4.1 答案



Erklärung des Codes:

Argument-Parsing:
    Der Code sucht die Position von | in den Argumenten, um die Argumente in zwei Teile aufzuteilen.
    argv wird entsprechend modifiziert, indem | durch NULL ersetzt wird.

Pipe-Erstellung:
    Mit pipe(pipe_fd) wird ein Rohr zwischen den beiden Programmen eingerichtet.

Erster Kindprozess:
    Der erste Kindprozess schließt das Leseende der Pipe und verbindet STDOUT_FILENO mit dem Schreibende der Pipe.
    Danach führt er das erste Programm mit execvp aus.

Zweiter Kindprozess:
    Der zweite Kindprozess schließt das Schreibende der Pipe und verbindet STDIN_FILENO mit dem Leseende der Pipe.
    Danach führt er das zweite Programm mit execvp aus.

Elternprozess:
    Der Elternprozess schließt beide Enden der Pipe und wartet darauf, dass beide Kindprozesse beendet werden.


Kompilieren Sie den Code mit: `gcc -o pipe pipe.c`
Beispielausführung:  `./pipe ls -la '|' head -n 5`,  Dies hat den gleichen Effekt wie ls -la | head -n 5 in der Shell.

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>

int main(int argc, char *argv[]) {
    if (argc < 4) {
        fprintf(stderr, "Usage: %s <program1> <args1> '|' <program2> <args2>\n", argv[0]);
        exit(EXIT_FAILURE);
    }

    // Find the position of '|'
    int pipe_pos = -1;
    for (int i = 1; i < argc; i++) {
        if (strcmp(argv[i], "|") == 0) {
            pipe_pos = i;
            break;
        }
    }

    if (pipe_pos == -1) {
        fprintf(stderr, "Error: Missing '|' in arguments\n");
        exit(EXIT_FAILURE);
    }

    // Replace '|' with NULL to split the arguments for execvp
    argv[pipe_pos] = NULL;

    // Create the pipe
    int pipe_fd[2];
    if (pipe(pipe_fd) == -1) {
        perror("pipe");
        exit(EXIT_FAILURE);
    }

    // Fork the first child process
    pid_t pid1 = fork();
    if (pid1 < 0) {
        perror("fork");
        exit(EXIT_FAILURE);
    }

    if (pid1 == 0) {
        // First child process
        close(pipe_fd[0]); // Close unused read end
        dup2(pipe_fd[1], STDOUT_FILENO); // Redirect stdout to pipe write end
        close(pipe_fd[1]); // Close the original write end

        // Execute the first program
        execvp(argv[1], &argv[1]);
        perror("execvp");
        exit(EXIT_FAILURE);
    }

    // Fork the second child process
    pid_t pid2 = fork();
    if (pid2 < 0) {
        perror("fork");
        exit(EXIT_FAILURE);
    }

    if (pid2 == 0) {
        // Second child process
        close(pipe_fd[1]); // Close unused write end
        dup2(pipe_fd[0], STDIN_FILENO); // Redirect stdin to pipe read end
        close(pipe_fd[0]); // Close the original read end

        // Execute the second program
        execvp(argv[pipe_pos + 1], &argv[pipe_pos + 1]);
        perror("execvp");
        exit(EXIT_FAILURE);
    }

    // Parent process
    close(pipe_fd[0]); // Close both ends of the pipe in the parent
    close(pipe_fd[1]);

    // Wait for both child processes to finish
    int status1, status2;
    waitpid(pid1, &status1, 0);
    waitpid(pid2, &status2, 0);

    return 0;
}
```


## 4.2 execvp


Die Funktion execvp ist eine der Funktionen der exec-Familie, die dazu dient, ein neues Programm im aktuellen Prozess auszuführen. Sobald execvp erfolgreich ausgeführt wurde, ersetzt das neue Programm den aktuellen Prozess. Der ursprüngliche Code nach execvp wird nicht mehr ausgeführt.

`int execvp(const char *file, char *const argv[]);`

Parameter:
1. **`file`**
    - Der Name des Programms, das ausgeführt werden soll.
    - Es kann entweder ein vollständiger Pfad sein (`/usr/bin/ls`) oder ein Programmnamen (`ls`), der im `PATH`-Umgebungsvariablen gesucht wird.
2. **`argv`**
    - Ein Null-terminiertes Array von C-Strings, das die Argumente für das Programm enthält.
    - `argv[0]` ist normalerweise der Name des Programms selbst.
    - Das Array muss mit `NULL` enden.

Rückgabewert:
- Bei Erfolg kehrt `execvp` **niemals** zurück, da der aktuelle Prozess durch das neue Programm ersetzt wird.
- Bei Fehler wird `-1` zurückgegeben, und `errno` wird gesetzt, um den Fehler zu beschreiben.


### 4.2.1 Beispiel 

```c
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>

int main() {
    // Programm und Argumente vorbereiten
    char *args[] = {"ls", "-la", NULL};

    // execvp ausführen
    if (execvp("ls", args) == -1) {
        perror("execvp failed");
        exit(EXIT_FAILURE);
    }

    return 0; // Dieser Code wird nicht ausgeführt, wenn execvp erfolgreich ist
}

```

Erklärung des Beispiels:
- Das Programm ruft `ls -la` auf.
- Wenn `execvp` erfolgreich ist, wird `ls` ausgeführt, und der Prozess wird durch das neue Programm ersetzt.
- Wenn `execvp` fehlschlägt (z. B. wenn `ls` nicht gefunden wird), gibt das Programm einen Fehler aus und beendet sich.


### 4.2.2 Vorteile von `execvp`

- Automatische Suche nach dem Programm in den Verzeichnissen, die in der `PATH`-Umgebungsvariablen definiert sind.
- Geeignet für dynamische und flexible Programmstarts, z. B. für Shell-ähnliche Programme.


### 4.2.3 Zusammenhang mit Pipes und Kindprozessen

- In Kombination mit `fork` wird `execvp` häufig verwendet, um in einem Kindprozess ein anderes Programm zu starten, während der Elternprozess weiterläuft.
- Beispiel für `fork` und `execvp`

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
    pid_t pid = fork();

    if (pid < 0) {
        perror("Fork failed");
        exit(EXIT_FAILURE);
    }

    if (pid == 0) { // Kindprozess
        char *args[] = {"ls", "-la", NULL};
        execvp("ls", args);
        perror("execvp failed");
        exit(EXIT_FAILURE);
    } else { // Elternprozess
        wait(NULL);
        printf("Child process completed.\n");
    }

    return 0;
}
```