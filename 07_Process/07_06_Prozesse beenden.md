
# 1 Der System Call `exit(status)` 

Der System Call `exit(status)` beendet den laufenden Prozess, schließt alle zugeordneten Dateien und gibt die vom Prozess belegten Systemressourcen frei.

Der Elternprozess wird mit dem [Signal](https://moodle.oncampus.de/modules/ir866/onmod/ipc/signals.html) SIGCHLD über das Ende des Kindprozesses benachrichtigt; ihm wird der `int`-Wert `status` als [Exit Status](https://moodle.oncampus.de/modules/ir866/onmod/shell/command-syntax.html#h_exit-status) des Kindes übergeben.
- 0 (in `stdlib.h` als Konstante `EXIT_SUCCESS` definiert) bedeutet erfolgreiche Beendigung
    - 0 就是结束 process 成功了
- jeder anderer Wert (oft 1 = `EXIT_FAILURE`) bedeutet einen Fehler

Beim Rücksprung aus der `main()`-Funktion wird automatisch `exit()` aufgerufen; der _exit status_ entspricht dem Rückgabewert. `return 0` in `main()` hat also den gleichen Effekt wie `exit(0)`.

## 1.1 Exit Handlers  `atexit()`

Mit der Funktion `atexit()` der Standard-Library (siehe [man 3 atexit](https://man7.org/linux/man-pages/man3/atexit.3.html)) kann man _exit handlers_ registrieren. Das sind Callback-Funktionen, die von `exit()` automatisch aufgerufen werden. Sie werden z.B. zum Aufräumen der nicht vom Kernel verwalteten Datenstrukturen von Systembibliotheken verwendet.

`_exit()` ist eine alternative exit-Funktion, bei der keine _exit handlers_ ausgeführt werden


## 1.2 Auf Kinder warten / Status abfragen `waitpid()`

Mit den Systemcall `waitpid()` kann ein Prozess auf die Beendigung eines Kindprozesses warten und dessen _exit status_ abfragen. Der aufrufende Prozess wird normalerweise blockiert, bis der Kindprozess über das Signal SIGCHLD seine Beendigung mitteilt.

- `waitpid(-1, wstatus, 0)` oder abgekürzt `wait(wstatus)` wartet auf _irgendein_ Kind. `wstatus` ist ein _Pointer_ auf einen `int`-Wert, in den der _exit status_ des Kindes und weitere Informationen geschrieben werden. Rückgabewert des Aufrufs ist die PID des Kindes.
- `waitpid(pid, wstatus, 0)` wartet auf einen den Kindprozess mit der angegebenen PID.

Mit dem Flag `WNOHANG` im 3. Argument wartet der Prozess nicht, sondern nur fragt nur die Statusinformation ab, falls ein Kind beendet wurde.

# 2 Zombies

> Elternprozess 再用 `waitpid()`  之后 , ein beendet Prozess 才会从  Prozesskontrollblock (PCB) 去除, 在这之前, 这个 beendete Prozess 一直都是 Zombie状态 

Der Prozesskontrollblock (PCB) eines beendeten Prozesses wird erst dann aus der Prozesstabelle entfernt, wenn der zugehörige Elternprozess dessen Status mit `waitpid()` abgefragt hat.

Bis dahin befindet sich der beendete Prozess in einem "untoten" Zustand, auch **Zombie** genannt. In der Ausgabe von `ps` sind Zombies am Zustand Z erkennbar.

1 
> _reaping_ 回收 
> eletern process 被结束了, 他的 kinderprocess 成为了 orphanned process, init-Prozess (PID 1) 会领养这个kinderprocess,   init-Prozess (PID 1) 会不断执行 wait() 去去除这个kinderprocess, 

Wenn ein Prozess _vor seinen Kindern_ terminiert, "adoptiert" standardmäßig der init-Prozess (PID 1) diese "Waisenkinder" (_orphaned processes_) und wird deren neuer Elternprozess. Der init-Prozess führt regelmäßig `wait()` aus, um verwaiste Zombies zu entfernen. Diesen Vorgang nennt man _reaping_.

Ein Prozess hat zwei Kinder und terminiert vor diesen. Welche Folge hat dies?
a. Die Kinder werden sofort zwangsweise beendet.
b. Die Kinder laufen weiter und werden später vom init-Prozess aufgeräumt; es entstehen keine Zombies.
c. Es entstehen zwei Zombies, die erst beim Neustart des Systems verschwinden.
选b 


2 
> System Call `prctl()` 去回收认养所有的orpaned processes 
In Linux kann sich ein Prozess mit dem Linux-spezifischen System Call `prctl(PR_SET_CHILD_SUBREAPER, 1, 0, 0, 0)` als _subreaper_ registrieren. Dann adoptiert dieser Prozess (und nicht PID 1) alle verwaisten Nachkommen seiner Kinder. Diese Möglichkeit wird z.B. unter Ubuntu zum Management von Login-Sessions mit `systemd` genutzt.



3
Zombies sollten normalerweise sehr schnell wieder verschwinden. Über längere Zeit im System existierende Zombies sind ein Hinweis auf fehlerhafte Programmierung, z.B. fehlendes `wait()`.

> Zombies 无法手动 manuell zu entfernen
Es gibt keine Möglichkeit, Zombies manuell zu entfernen. Solange der Elternprozess ihren Status nicht mit `waitpid()` abfragt, bleiben sie bis zum Neustart des Systems erhalten.


> Zombies基本是无害的 
Da Zombies nur Einträge in der Prozesstabelle belegen und sonst keine Systemressourcen verbrauchen, sind sie aber harmlos, solange sie nicht in sehr großer Zahl auftreten.



# 3 fork - exec - wait in Kombination

Oft werden `fork()`, `execve()` und `wait()` miteinander kombiniert, um ein Programm synchron in einem Kindprozess auszuführen:

1. mit `fork()` einen Kindprozess erzeugen
    
2. mit `execve()` im Kind das Kommando ausführen
    
3. mit `wait()` auf das Kind warten
    
4. ggf. dessen Ergebnisse auswerten
    

Diese Kombination wird z.B. verwendet, um in der Shell Kommandos im Vordergrund auszuführen.
