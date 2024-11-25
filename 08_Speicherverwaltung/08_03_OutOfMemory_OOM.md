
储存空间满了, 没空了 

Eine **Out of Memory (OOM)** Situation tritt ein, wenn das System zur Laufzeit angeforderten Speicher nicht bereitstellen kann, weil RAM und Swap Space voll sind.

In diesem Fall startet der Kernel den **OOM killer** — eine Funktion, die einen Prozess auswählt und beendet, um so Speicher freizugeben. Der Algorithmus sucht einen Prozess aus, der viel Speicher belegt, aber noch nicht so lange läuft.


OOM ist ein ernster Fehler, der zu unerwünschtem Systemverhalten und Datenverlust führen kann und im Normalbetrieb nie auftreten sollte. Gehen Sie daher jeder OOM-Situation nach und versuchen Sie die Ursachen zu ermitteln und zu beheben. 

Typische Ursachen sind (oft in Kombination):
- Zu wenig bereitgestellter Swap Space
- Unüblich hohe Systemlast
- Fehlende oder zu hoch eingestellte Ressourcen-Limits für einzelne User/Prozesse
- Fehler der Anwendungssoftware: unnötig Speicher reseviert, _memory leaks_ in lang laufenden Prozessen1