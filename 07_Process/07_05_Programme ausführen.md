
Mit dem System Call `execve()` kann ein Prozess ein neues Programm aus einer Datei laden und ausführen. Das neue Programm **ersetzt** das bisherige Programm. Offene _file descriptors_ bleiben aber erhalten. Die Parameter von `execve()` sind:

1. Der Pfad des auszuführenden Programms
    1. `execve()` verwendet _nicht_ den Suchpfad PATH, daher müssen Sie einen absoluten oder relativen Pfad zum auszuführenden Programm angeben.
2. Argumente, die als `argv[]` an die `main()`-Funktion des neuen Programms übergeben werden
3. Das [Environment](https://moodle.oncampus.de/modules/ir866/onmod/proc/environment.html) für das neue Programm.

execve.c
```c
#include <stdio.h>     // for perror()
#include <unistd.h>    // for execve()
extern char **environ; // current process environment

int main() {

  char *args[] = { "ls", "-l", "/usr", NULL }; 

  execve("/usr/bin/ls", args, environ); 
  // should not get here unless execve failed
  perror("execve failed"); 
}
```

|   |   |
|---|---|
|![1](https://moodle.oncampus.de/modules/ir866/onmod/icons/callouts/1.svg)|Erstes Argument ist per Konvention der Name des Programms selbst. Die Argument-Liste endet mit einem NULL-Pointer.|
|![2](https://moodle.oncampus.de/modules/ir866/onmod/icons/callouts/2.svg)|Das neue Programm verwendet das bestehende Environment.|
|![3](https://moodle.oncampus.de/modules/ir866/onmod/icons/callouts/3.svg)|`execve()` kommt nur im Fehlerfall zurück, der return-Wert ist immer -1 und muss hier nicht abgefragt werden.|

1. Übersetzen und testen Sie das Beispiel `execve.c`.
    
2. Was passiert, wenn Sie statt `/usr/bin/ls` nur `ls` aufrufen?
    
3. Ändern Sie das Programm ab, um statt `ls` das Programm `printenv` ohne Argumente auszuführen. Damit sollte das aktuelle Environment ausgegeben werden.
    
4. Konstruieren und verwenden Sie ein neues Environment, das nur aus der Variablen `HALLO=welt` besteht. Jetzt sollte `printenv` nur diese eine Variable anzeigen.



POSIX definiert einige Varianten von `execve()`, die für manche Zwecke komfortabler zu verwenden sind, z.B. `execvp()`, das wie die Shell das auszuführende Programm in PATH sucht. Diese Funktionen sind in der C-Standard-Library implementiert und werden intern auf den System Call `execve()` abgebildet. Kollektiv bezeichnet man sie als `exec()`-_Familie_.


# 1 小题目 


Welche Aussagen zu den Funktionen der **exec()**-Familie sind korrekt?  

a. Der Rückgabewert von **execle()** ist der _exit status_ des ausgeführten Programms.  
falsch
The return value of `execle()` (or any `exec()` function) is not the exit status of the executed program. These functions do not return on success. They only return `-1` on failure, and the `errno` variable is set to indicate the error.


b **execve()** kann das Environment des Prozesses neu setzen.  
richtig 
`execve()` can set the environment of the process. This function allows specifying a new environment for the executed program by passing an explicit environment array as one of its arguments.


c. **execve()** benötigt den absoluten Pfad des auszuführenden Programms.  
falsch 
`execve()` does not strictly require an absolute path. It can execute a program using a relative path, but the path must be correct relative to the current working directory.


d. **execle()** und **execve()** unterscheiden sich nur darin, wie die Argumentliste des neuen Programms übergeben wird.  
richtig  
`execle()` and `execve()` differ not just in how arguments are passed but also in how they handle the environment. `execle()` allows passing a custom environment, while `execve()` explicitly requires both arguments and environment to be provided.


e. Programmcode nach einem exec() Aufruf wird nur im Fehlerfall ausgeführt. 
richtig 
Code after an exec() call is executed only if the exec() call fails. On success, the exec() family replaces the calling process image with a new program, and no subsequent code in the original process is executed.


f. Shell Scripts können nicht mit den Aufrufen der **exec()**-Famile gestartet werden.  
falsch 
Shell scripts **can** be started with the `exec()` family, provided the script starts with a valid shebang (`#!`) line specifying the interpreter (e.g., `#!/bin/bash`). If no shebang is present, the execution depends on the system's behavior or defaults.


g. **execvp()** sucht in PATH nach dem ausführbaren Programm.
richtig 
`execvp()` searches for the executable program in the `PATH` environment variable if a relative path or program name is provided.



