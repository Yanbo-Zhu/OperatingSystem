
Prozess 接收到一个 Signal 

**Signale** sind eine primitve, aber sehr effiziente Form der asynchronen IPC in Unix-Systemen. Der Kernel kennt einen fest definierten Satz möglicher Signale, die gesendet werden können. 
Jedes Signal hat einen symbolischen Namen und eine Nummer (z.B. SIGKILL mit der Nummer 9). POSIX standardisiert 28 Signale und Ihre Bedeutung, nicht aber die Nummern.


kill 命令 不是用来 结束语一个 prozess, 而是用来发送 signal 给某个 proizess, 这个 signal 可以是任何
Mit dem System Call [kill()](https://man7.org/linux/man-pages/man2/kill.2.html) oder dem Shell-Kommando `kill` kann ein Prozess ein Signal an einen anderen Prozess oder eine ganze Prozessgruppe senden. Unprivilegierte Prozesse können Signale nur an Prozesse mit der gleichen User-ID senden.

Der empfangende Prozess erfährt, welches Signal gesendet wurde und von wem (PID des Absenders). Weitere Informationen können nicht übertragen werden. Es gibt keine Rückmeldung an den Absender, ob und wann ein gesendetes Signal vom Empfänger behandelt wurde.

# 1 Signal Handling

Ein Prozess entscheidet anhand der Signalnummer, wie ein empfangenes Signal behandelt wird. 

Dafür gibt es drei prinzipielle Möglichkeiten:
1. Das Signal _annehmen und behandeln_; dann wird eine hinterlegte _signal handler_-Funktion automatisch ausgeführt. Jedes Signal hat eine Default-Aktion, z.B. den Prozess zu beenden. Diese kann durch eine individuelle _signal handler_-Funktion ersetzt werden.
2. Das Signal _blockieren (maskieren)_; dann wird das Signal gespeichert und später behandelt, wenn die Blockierung aufgehoben wird.
3. Das Signal _ignorieren_; dann passiert nichts.

Mit dem System Call [sigaction()](https://man7.org/linux/man-pages/man2/sigaction.2.html) kann ein Prozess festlegen, welche dieser Aktionen für ein bestimmtes Signal ausgeführt wird.

Es ist auch möglich, auf ein Signal zu warten: [pause()](https://man7.org/linux/man-pages/man2/pause.2.html) blockiert den Prozess, bis ein Signal ankommt.

Dieses Beispiel zeigt, wie Signale ignoriert oder behandelt werden:

signal-handler.c
```c
#include <stdio.h>
#include <signal.h>
#include <unistd.h>

void myhandler (int sig) {
  printf("Received signal %d, but I don't care!\n", sig);
}

int main() {
  struct sigaction ignoreme, handleme;
  ignoreme.sa_handler = SIG_IGN;
  handleme.sa_handler = myhandler;

  printf("my PID is %d\n", getpid());

  sigaction(SIGTSTP, &ignoreme, NULL);  // ignore Ctrl-Z
  sigaction(SIGINT, &handleme, NULL);   // handle Ctrl-C

  while(1) pause();  // suspend process until signal received
}
```


Bauen und testen Sie das Programm.
1. Was passiert, wenn Sie Ctrl+c oder Ctrl+z drücken?
2. Wie können Sie das Programm aus einer anderen Shell beenden?
3. Mit welcher Tastatureingabe können Sie das Programm beenden?

Antwort
1. Ctrl+c gibt die Meldung _Received signal 2, but I don’t care!_ aus; Ctrl+z wird ignoriert.
2. `kill -9 <pid>`
3. Ctrl+\ (sendet das Signal SIGQUIT)


# 2 Wichtige Signale

Die folgende Tabelle enthält eine Auswahl wichtiger Signale zur Steuerung von Prozessen:

|   |   |
|---|---|
|SIGINT|_interrupt from keyboard_: Ctrl-C|
|SIGTSTP|_stop typed at terminal_: Ctrl-Z|
|SIGTERM|_terminate_: Prozess beenden, kann für "Aufräumarbeiten" behandelt werden|
|SIGKILL|_kill_: Prozess sofort und "hart" beenden, hat traditionell Nummer 9 (`kill -9`)|
|SIGSTOP|_stop_: Prozess stoppen|
|SIGCONT|_continue_: Gestoppten Prozess fortsetzen|

SIGKILL und SIGSTOP funktionieren immer, sie können weder ignoriert noch individuell behandelt werden. SIGKILL (kill -9) ist das "letzte Mittel" zum Beenden von Prozessen, die nicht mehr reagieren.

Einige Signale werden direkt vom Kernel gesendet oder aus Hardware-Events generiert, z.B.

|   |   |
|---|---|
|SIGCHLD|_child_: Kindprozess terminiert|
|SIGFPE|_floating point exception_: fatale arithmetische Fehler wie eine Division durch 0|
|SIGSEGV|_segmentation violation_: Zugriff auf eine unerlaubte Speicheradresse (Bug!)|

Das Kommando kill -L gibt alle in Ubuntu verfügbaren Signale aus.

