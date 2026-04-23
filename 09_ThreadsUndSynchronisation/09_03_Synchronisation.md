Prozesse oder Threads 共同使用 Ressource 
Wenn nebenläufig ausgeführte Prozesse oder Threads gemeinsame Ressourcen nutzen, kann es sein, dass das Ergebnis von der Reihenfolge der Ausführung abhängt. Diese Situation nennt man ==**Race Condition**==. Es kann zu inkorrekten Resultaten oder zu Laufzeitfehlern bis hin zum Absturz der Anwendung kommen.

 **kritischer Abschnitt** 就是 Prozesse oder Threads 共同使用 Ressource  导致的 一个使用的交叉 
Ein **kritischer Abschnitt** eines nebenläufigen Programms ist ein Programmteil, in dem zu jeder Zeit höchstens ein Prozess oder Thread sein darf, um Race Conditions zu vermeiden. Oft besteht ein kritischer Abschnitt aus mehreren Codeabschnitten, z.B. allen, die auf eine gemeinsame globale Variable zugreifen.

Race Conditions treten gerade mit schnellen Rechnern oft nur sporadisch auf, da sich dann die beteiligten Threads jeweils nur sehr kurz im kritischen Abschnitt befinden. Sie sind daher oft schwer zu reproduzieren. Auch das Debugging ist schwierig: Debugging-Methoden wie Breakpoints ändern das zeitliche Verhalten der Anwendung, sodass die Race Condition evtl. nicht mehr oder anders erscheint.

Race conditions können Ihren Code verwundbar (使受伤,使负伤) machen: Wenn es einem Angreifer gelingt  成功，达到预定目的, die race condition zu provozieren, z.B. über spezielle Inputs oder einfach durch häufige Wiederholung einer Aktion, dann kann er das fehlerhafte Systemverhalten vielleicht für seine Zwecke ausnutzen.
If an attacker manages to exploit the race condition, for example, through specific inputs or simply by repeatedly performing an action, they may be able to take advantage of the faulty system behavior for their own purposes.

# 1 Beispiel einer Race Condition

Betrachten Sie das Programm `racecond.c`. Es inkrementiert einen globalen Zähler (mit Startwert 0) in zwei Threads je 500-mal. Am Ende sollte der Zähler den Wert 1000 haben. 

Im kritischen Abschnitt wird die globale Variable `counter` gelesen, inkrementiert und zurückgeschrieben. Normalerweise würde man im Programm dafür einfach `counter++;` schreiben. Dann würde aber die Compiler-Optimierung die Schleife wegoptimieren und in einem Schritt `counter` um 500 erhöhen. Selbst wenn man die Optimierung deaktiviert, ist der kritische Abschnitt zu schnell, um eine Race Condition zuverlässig reproduzieren zu können.
(
In the critical section, the global variable `counter` is read, incremented, and written back. Normally, you would simply write `counter++;` in the program for this. However, compiler optimization would then optimize away the loop and increase `counter` by 500 in a single step. Even if optimization is disabled, the critical section is too fast to reliably reproduce a race condition.
compiler optimization 会自动加500, 省略中间的累计的每一步 
)


Zur Demonstration der Race Condition verlangsamen wir den kritischen Abschnitt künstlich durch eine zufällige Verzögerung, deren maximale Länge durch ein Argument des Progamms parametrierbar ist. Dadurch kommt auch die Compiler-Optimierung nicht mehr zum Zug:

```c
int x = counter;
x++;

// random delay
int r = random() % (1<<delay);
for (int j=0; j<r; j++);

counter = x;

```

Je nach Länge der Verzögerung ist es mehr oder weniger wahrscheinlich, dass ein Thread zwischen dem Lesen von `counter` und dem Zurückschreiben des inkrementierten Werts von dem anderen unterbrochen wird. Dann wird später ein veralteter Wert zurückgeschrieben und es gehen Inkrementierungen verloren.

## 1.1 
Übersetzen Sie das Programm `racecond` und testen Sie es mit verschiedenen Werten des Arguments `delay`. Welche Auswirkung hat die Länge der Verzögerung auf das Ergebnis?

Unkommentieren Sie jetzt den Aufruf von `use_single_cpu()`. Diese Funktion weist den Kernel an, beide Threads auf derselben CPU laufen zu lassen. Testen Sie das Programm erneut mit verschiedenen `delay`-Werten und erklären Sie die beobachteten Veränderungen. Kommentieren Sie danach `use_single_cpu()` wieder aus.



# 2 Thread-safe Funktionen
variable oder Funktion  总是同时被多个 theards oder prozessen aufgerufen
wenn ein variable oder Funktion  **thread-safe**  ist,  那他怎么被 aufgerufen 都没事 

Dieselbe Bibliotheksfunktionen wird oft aus mehreren Threads oder Prozessen aufgerufen. Dies sollte nicht zu Problemen führen.

> Eine Funktion heißt **thread-safe**, wenn Sie gleichzeitig aus verschiedenen Threads aufgerufen werden kann und immer das korrekte Ergebnis liefert, egal wie die Threads zeitlich ausgeführt werden.

Um dies zu erreichen, ==müssen kritische Zugriffe synchronisiert werden== 要使得 每次的 Zugriff 相互之间都被同步 . Problematisch sind dabei vor allem globale Variablen. Lokale Variablen sind an sich _thread-safe_, da jeder Thread einen eigenen Stack hat. Sie können aber Referenzen (Pointer) auf Objekte enthalten, die mehr als einem Thread bekannt sind.

Der POSIX-Standard verlangt, dass [die meisten dort definierten Bibliotheksfunktionen](https://pubs.opengroup.org/onlinepubs/9699919799/functions/V2_chap02.html#tag_15_09) _thread-safe_ sind. System Calls müssen immer _thread-safe_ sein, sonst würde das Betriebssystem nicht funktionieren.

Ob Library-Funktionen _thread-safe_ sind, ist üblicherweise in der Dokumentation (z.B. man page) erwähnt.
