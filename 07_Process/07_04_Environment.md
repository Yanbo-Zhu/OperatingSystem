

Jeder Prozess in Unix hat ein **Environment**. Dies besteht aus einer Menge von Variablen.  
Die Werte sind jeweils Strings.  
Die Variablennamen bestehen aus Großbuchstaben, Ziffern, und "`_`" ; das erste Zeichen darf keine Ziffer sein.

In C-Programmen ist das gesamte Environment eines Prozesses als _globale Variable_ zugänglich:

```c
extern char **environ;
```


Diese Variable ist ein unsortiertes, NULL-terminiertes Array von Zeigern auf Strings der Form "VARIABLE=WERT". Um direkt darüber auf eine bestimmte Environment-Variable zuzugreifen, muss man erst nach dem Namen suchen und dann aus dem String "VARIABLE=WERT" den Wert extrahieren.

Weil dies sehr umständlich wäre, definiert der POSIX-Standard _Zugriffsfunktionen_, die in der Standard-Library implementiert werden:

|   |   |
|---|---|
|`getenv(name)`|gibt den Wert der Variablen zurück (oder `NULL`, falls nicht definiert).|
|`setenv(name, value, 1)`|Weist der Variablen den Wert `value` zu|
|`setenv(name, value, 0)`|Zuweisung erfolgt nur, wenn die Variable noch keinen Wert hat|
|`unsetenv(name)`|Löscht die Variable aus dem Environment|

Bei `fork()` wird das Environment an den neuen Kindprozess vererbt; dieser erhält eine _identische Kopie_. Spätere Änderungen des Environments im Eltern- oder Kindprozess sind unabhängig voneinander.

Über Environment-Variablen werden typischerweise Informationen und Einstellungen verteilt, die für eine Gruppe von Prozessen relevant sind, z.B. alle Prozesse einer Benutzer-Session. Beispiele sind:

- `EDITOR`: vom Benutzer festgelegter Default-Texteditor
    
- `HOME`: Home-Directory des aktuellen Benutzers
    
- `PATH`: Suchpfad für ausführbare Programme
    
- `TERM`: Typ des zugeordneten Terminals
    
- `USER`: aktueller User

Sehen Sie sich mit das aktuelle Environment Ihrer Shell an: `printenv | sort`. Recherchieren Sie exemplarisch, was einige der Variablen bedeuten.


# 1 小题目 


a. Ein Prozess und seine Kinder teilen sich ein gemeinsames Environment, auf das sie alle schreibend und lesend zugreifen können.
_Incorrect_  
A process and its children do not share the same environment. Each child process receives a copy of the parent's environment during the `fork()` call. Changes to the environment of one process do not affect the environment of another process.


b.
Schreibt ein Prozess eine Environment-Variable, so wird die Änderung automatisch an alle existierenden Kinder des Prozesses weitergeben, nicht aber an seinen Elternprozess.

Incorrect
Changes to the environment of a process are not propagated to its children or parent. Each process has its own separate copy of the environment.


c. Im Aufruf von fork() kann ein geändertes Environment für das Kind übergeben werden.
_Incorrect_  
The `fork()` system call does not allow passing a different environment to the child process. The child inherits a copy of the parent's environment by default. To set a new environment, an `exec()` family function must be used after `fork()`.


d. Bei fork() wird das Environment als unabhängige Kopie an den Kindprozess vererbt.

_Correct_  
When `fork()` is called, the child process receives an independent copy of the parent's environment. Changes made by one process to its environment do not affect the other.


e. Der Wert einer Environment-Variablen kann ein String sein.
Correct
Environment variables are represented as strings in the format NAME=value.

f. Der Wert einer Environment-Variablen kann eine Integer-Zahl sein.
Incorrect
Environment variables are always strings. Even if a variable contains a numeric value (e.g., VAR=42), it is still treated as a string.


# 2 #
Wie viele Prozesse werden erzeugt?  

Sie führen folgendes Programmstück aus:  

```c
int main() { 
  if (fork() == 0) { // First fork
    fork();          // Second fork
    fork();          // Third fork
  }
  printf("hallo\n");
  sleep(5);
}
```

Laufzeitfehler kommen dabei nicht vor. Wie oft wird "**hallo**" ausgegeben?  
Geben Sie Ihre Antwort als Zahl ein.

Processes Created: 5 processes (P0, P1, P2, P3, P4).
Times "hallo" Is Printed: 5 times (once by each process).

fork() 被执行后, 会打开一个新的process, 并且从该fork() 后面的语句开始执行 

---

**Initial Process (P0):**
The program starts with **1 process**, called `P0`.


**Step 1: The First `fork()`**
- `fork()` creates a new process.
- In the parent process (`P0`), `fork()` returns a **non-zero** value. This means the parent process skips the code inside the `if` block and goes to the `printf` statement.
- In the child process (`P1`), `fork()` returns **0**. This means the child process enters the `if` block.
At this stage:
- **Processes:** `P0` and `P1` (2 processes total).


**Step 2: The Second `fork()` (Inside `P1`):**
- Only `P1` executes this `fork()` because it entered the `if` block.
- `P1` creates a new process (`P2`).

At this stage:
- **Processes:** `P0`, `P1`, and `P2` (3 processes total).
- `P1` and `P2` continue executing from the line after the second `fork()`.

 
 **Step 3: The Third `fork()` (Inside `P1` and `P2`):**
- Both `P1` and `P2` execute this `fork()`.
- `P1` creates a new process (`P3`).
- `P2` creates a new process (`P4`).
At this stage:
- **Processes:** `P0`, `P1`, `P2`, `P3`, and `P4` (5 processes total).
- All five processes (`P0`, `P1`, `P2` `P3`, and `P4`) now proceed to the `printf` statement.

 
 **Step 4: Printing "hallo"**
- Every process executes `printf("hallo\n");` because there are no conditions preventing it.
- Each process prints `"hallo"` exactly once.

At this stage:
- The word `"hallo"` is printed **5 times**.



Here is a visual representation of how processes are created:
P0
 ├── P1 (First fork, enters `if`)
 │    ├── P2 (Second fork)
 │    │    └── P4 (Third fork, by P2)
 │    └── P3 (Third fork, by P1)

- **P0**: Executes `printf` and sleeps.
- **P1**: Executes `fork()` twice, then `printf` and sleeps.
- **P2**: Executes one `fork()`, then `printf` and sleeps.
- **P3**: Executes `printf` and sleeps.
- **P4**: Executes `printf` and sleeps.