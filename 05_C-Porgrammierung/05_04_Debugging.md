
Die Fehlersuche in C-Programmen kann ziemlich herausfordernd sein: Fehlermeldungen des Compilers sind oft kryptisch. Bei Laufzeitfehlern stürzt das Programm mit wenig hilfreichen Meldungen wie _segmentation fault_ ab; es gibt weder Exceptions noch einen Stacktrace wie bei Java.

Es gibt verschiedene Methoden zum Debuggen von C-Programmen, einige davon stellen wir hier kurz vor.


# 1 Debugging-Ausgaben


Fügen Sie im Quellcode gezielt Anweisungen ein, die Informationen über den Stand der Bearbeitung ausgeben, z.B. Variablenwerte oder welcher Teil der Bearbeitung gerade erfolgt. 
Ausgaben sollten auf stderr erfolgen, um den normalen Output nicht zu stören.

Mit der Präprozessor-Direktive `#ifdef` können Sie den Debugging-Code bei der Compilierung zentral aktivieren oder deaktivieren:

```sh
#define DEBUG 
...
#ifdef DEBUG 
  fprintf(stderr,"Wert von i in foo() = %d\n", i);
#endif
...
```

Die Macro-Definition aktiviert den Debugging-Code in der ganzen Datei; zum Deaktivieren löschen.
Codestück wird nur übersetzt, wenn DEBUG definiert ist.


# 2 Debugger (GDB )

Sie können einen Debugger verwenden. Mit diesem separaten Tool können Sie z.B.

- Breakpoints im Quelldcode setzen
    
- Variablenwerde beobachten
    
- Schrittweise einzelne Anweisungen im C- oder Assemblercode ausführen (_stepping_).
    

[GDB](https://www.sourceware.org/gdb/), der _GNU Project Debugger_, ist ein klassischer Debugger für C- und Assemblercode auf Unix-Systemen. GDB bietet eine textbasierte Benutzeroberfläche sowie ein API zur Steuerung aus anderen Applikationen. Viele grafische IDEs verwenden intern GDB und sein API für das Debugging.

Installieren Sie GDB auf Ihrem Ubuntu-System: `sudo apt install gdb`.



# 3 Tracing: strace und ltrace


Unter _tracing_ versteht man die Nachverfolgung der Aktivitäten eines Programms zur Laufzeit.

Das Tool `strace` (in Ubuntu-Standardinstallation enthalten) führt ein gegebenes Programm aus und protokolliert dabei alle von diesem Programm aufgerufenen System Calls, deren Argumente und Rückgabewerte. Es hängt sich quasi in die System-Call Schnittstelle ein und liest die Kommunikation des Programms mit dem Kernel mit.

Lesen Sie `man strace` und probieren Sie es mit `hello` aus:
`strace ./hello`  

Was können Sie im Output von `strace` erkennen?


`ltrace` arbeitet ähnlich wie `strace`, protokolliert aber Aufrufe in dynamisch geladene Libraries anstelle von System Calls.

