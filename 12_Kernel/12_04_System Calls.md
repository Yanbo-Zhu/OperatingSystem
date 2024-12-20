
> Ein **System Call** (kurz _syscall_) ist eine standardisierte Schnittstelle, über die Anwendungsprogramme Dienste des Kernels aufrufen können. Jeder Unix-Kernel stellt einen festen Satz an System Calls bereit.

Die System Calls ermöglichen eine saubere Trennung zwischen Kernel und Anwendungsprogrammen: Der Wechsel vom unprivilegierten _user mode_ in den privilegierten _kernel mode_ und zurück ist nur über spezielle CPU-Instruktionen möglich.

Anwendungen können daher im _kernel mode_ ausschließlich die vordefinierten Funktionen der System Calls nutzen. So wird verhindert, dass beliebiger Anwendungs-Code im _kernel mode_ ausgeführt wird.

Auch ein privilegierter "root-Prozess" läuft im _user mode_ und muss die System Calls nutzen. Er unterliegt bei der Ausführung von System Calls "nur" weniger Einschränkungen als normale Prozesse.

Jeder System Call
- bietet eine klar spezifizierte, relativ einfache Funktionalität;
- stellt die Korrektheit der Anforderung sicher und überprüft ggf. die Berechtigung des Aufrufers;
- gibt einen Rückgabewert zurück, der insbesondere erkennen lässt, ob die Ausführung erfolgreich war.


---


Auch wenn der grundsätzliche Ablauf immer ähnlich ist, hängen einige Details des System Call-Mechanismus von der Hardware-Architektur und der konkreten Implementierung des Kernels ab.  
Dazu gehören insbesondere:
- Konventionen zur Übergabe von System Call-Nummern, Parameter und Rückgabewerte: über Register (welche?) und/oder den Stack?
- Instruktionen zum Wechsel in den Kernel Mode und wieder zurück
- Mechanismen zum Sichern und Wiederherstellen des Hardware-Kontexts

Um Anwendungsprogrammierer von solchen Details zu entlasten und möglichst portablen Quellcode zu erhalten, werden System Calls im Normalfall von der Anwendung nicht direkt (per Trap-Instruktion) aufgerufen. ==Stattdessen verwendet man entsprechende **Funktionen der C-Standardbibliothek**,== die eine portable Schnittstelle bieten und systemspezifisch in Assemblercode implementiert sind.


# 1 Prinzipieller Ablauf

Das folgende Bild zeigt den prinzipiellen Ablauf eines System Calls am Beispiel `close(3)`. Wir nehmen an, dass der System Call die Nummer 10 hat.
执行完 close(3)  这个 function 后  发生的一些操作 
 der System Call die Nummer 10 hat. 所以 第2步中 mov eax 后面为 10

![](images/Pasted%20image%2020241220205555.png)

1. **Parameter** werden in Registern / auf dem Stack übergeben und die Bibliotheksfunktion für den System Call aufgerufen.
2. Die Bibliotheksfunktion legt die **Nummer des System Calls** in ein Register.
3. Dann wird eine **Trap-Instruktion** (Software-Interrupt) ausgeführt, die den Hardware-Kontext (Register, CPU-Status etc.) auf dem Stack sichert und in den _kernel mode_ wechselt. 
4. Der Kernel erkennt anhand der Trap-Nummer, dass es sich um einen System Call handelt, und startet den **Dispatcher** (eine Verteil-Funktion).
5. Der Dispatcher sucht in **_system call table_** anhand der System Call-Nummer (10) die passende Systemroutine und ruft sie auf.
6. Die **Systemroutine** (=eigentliche Implementierung des System Calls) prüft Parameter und Berechtigung des Aufrufers und bearbeitet die Anforderung.
7. Die Ergebnisse (Rückgabewerte) des System Calls werden in Registern oder auf dem Stack bereitgestellt.
8. Mit einer speziellen Instruktion kehrt die CPU in den _user mode_ zurück; dabei wird der vorher gesicherte Hardware-Kontext wiederhergestellt.
9. Das Anwendungsprogramm läuft weiter und kann das Ergebnis des System Calls abfragen.

## 1.1 System Calls in Linux

Moderne Linux-Versionen kennen über 300 System Calls. Die genaue Anzahl hängt von der CPU-Architektur und der Kernel-Konfiguration ab.

[syscalls(2)](https://man7.org/linux/man-pages/man2/syscalls.2.html) enthält eine Liste der System Calls von Linux. Sehen Sie die Liste durch. Recherchieren Sie exemplarisch die Funktionalität einiger System Calls, die Sie _noch nicht_ kennen.

Falls Sie sich für die Implementierung von Linux-System Calls interessieren, empfehle ich Ihnen diese [schöne suchbare Tabelle](https://filippo.io/linux-syscall-table/) mit Link zur Implementierung im Quellcode.


