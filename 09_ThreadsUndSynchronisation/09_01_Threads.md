
**Threads** sind "leichtgewichtige Prozesse": Ausführungseinheiten, die _innerhalb_ eines Prozesses nebenläufig verarbeitet werden und viele Ressourcen des Prozesses teilen.

Threads eines Prozesses führen das gleiche Programm aus, teilen sich den ==gleichen globalen Speicher== (Datensegment und Heap) und auch die gleichen offenen Dateien (_file descriptors_). ==Sie sind also — im Gegensatz zu separaten Prozessen — nicht gegeneinander geschützt.==

Jeder Thread hat aber einen eigenen _call stack_ für lokale Variablen und Rücksprungadressen, um einen separaten Kontrollfluss zu ermöglichen.

# 1 Wozu?

Threads verwendet man, um die Arbeit innerhalb eines Prozesses aufzuteilen und zu parallelisieren. So kann man die Performance der Anwendung deutlich erhöhen und die Software übersichtlicher strukturieren.

Beispielsweise könnte bei einem Textverarbeitungsprogramm
- ein Thread auf Input des Anwenders warten
- einer die Formatierung und Anzeige auf dem Bildschirm übernehmen
- ein dritter im Hintergrund die laufende Rechtschreibprüfung machen.

Das alles könnte man prinzipiell auch mit getrennten Prozessen erledigen. Da diese aber keine gemeinsamen Ressourcen haben, müssten sie über [spezielle Mechanismen](https://moodle.oncampus.de/modules/ir866/onmod/ipc.html) kommunizieren. Das ist umständlicher zu programmieren als mit Threads und benötigt auch mehr Systemressourcen.

# 2 Kernel- und User-level Threads

Es gibt zwei grundsätzliche Ansätze, Threads zu implementieren. Sie unterscheidet sich darin, wo im System die Threads verwaltet werden:

> **Kernel-level Threads** werden vom Kernel selbst verwaltet. Der Kernel stellt der Anwendung System Calls zum Erzeugen, Beenden etc. von Threads zur Verfügung und übernimmt das Thread-Scheduling. 


> **User-level Threads** werden von einer ins Anwendungsprogramm eingebundenen Thread-Bibliothek im _user mode_ verwaltet. Der Kernel sieht nur den Prozess als Ganzes und "weiß nichts" von den Threads.  将 prozess 视为一个整体, 根本不知道其中的 Threads 

Beide Ansätze haben Vor- und Nachteile:
- _Kernel-level Threads_ können vom Kernel auf verschiedene CPUs abgebildet werden. Wenn ein Thread (z.B. für I/O) blockiert, können die anderen weiterlaufen. Dafür erfordern Thread-Kontextwechsel einen System Call und sind weniger effizient.
- _User-level threads_ ermöglichen sehr schnelle Thread-Kontextwechsel und belegen keine Kernel-Ressourcen. Aber sie können nur eine CPU nutzen. Wenn ein Thread blockiert, blockiert der ganze Prozess.

In modernen Multiprozessor-Systemen werden _vorwiegend Kernel-level Threads_ verwendet, weil man damit mehrere CPUs nutzen kann. Es gibt auch Mischformen, bei denen User-level Threads verwendet, aber "m:n" auf mehrere Kernel-level Threads aufgeteilt werden.


## 2.1 为什么 Wenn ein Thread blockiert, blockiert der ganze Prozess.
Angenommen, Sie verwenden User-level Threads und ein Thread blockiert, weil er auf eine Eingabe aus einer Datei wartet. Warum kann dann nicht ein anderer Thread weiterarbeiten, der die gelesenen Daten gar nicht benötigt?
一个 User-level Threads  在等  eine Eingabe aus einer Datei, 这时候 ganze Prozess wurde blockiert und alle anderen Threads des Process wurde auch blockiert, 为什么 

Antwort
Die Dateieingabe wird über einen System Call vom Kernel angefordert. Der Kernel kennt die User-level Threads der Anwendung nicht und muss daher den aufrufenden Prozess komplett blockieren, bis das Ergebnis verfügbar ist.

Dateieingabe 通过 System Call vom Kernel  被提出. Therads 通过 prozess 做一个 systemaufruf , 为了从 kernel 中得到数据 . 
对于 User-level Threads , Kernel不知道 那个 Threads der Anwendung. 他只能阻碍整个process, 直到 Dateioperation 被完成. 然后再 放开整个process 

Wenn User-Level-Threads verwendet werden, handelt es sich um Threads, die vom Benutzerprozess verwaltet werden und nicht direkt vom Betriebssystem. Der Kernel kennt die Threads auf Benutzerebene nicht und kann daher nicht erkennen, dass andere Threads innerhalb des Prozesses weiterhin ausgeführt werden könnten.

In dem beschriebenen Szenario, wenn ein Thread auf eine Eingabe aus einer Datei wartet, führt er in der Regel einen Systemaufruf (System Call) aus, um die Daten vom Kernel anzufordern. Der Kernel blockiert dann den gesamten Prozess, bis die Dateioperation abgeschlossen ist, weil der Kernel keine Kenntnis von den einzelnen Threads auf Benutzerebene hat. Der gesamte Prozess wird blockiert, einschließlich aller anderen Threads, die vielleicht nicht auf die Datei zugreifen, da der Kernel keine Möglichkeit hat, den Blockierungsstatus einzelner User-Level-Threads zu erkennen.

Deshalb kann in diesem Fall kein anderer Thread weiterarbeiten, auch wenn er die gelesenen Daten nicht benötigt.

