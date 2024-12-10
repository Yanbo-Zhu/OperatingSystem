
Eine **FIFO (named pipe)** ist eine _persistent_ angelegte Pipe, die im Dateisystem als Datei erscheint. 
Im Output von `ls -l` ist sie am Typ `p` erkennbar.

Eine FIFO wird angelegt
- aus der Shell mit dem Kommando `mkfifo` ([mkfifo(1)](https://man7.org/linux/man-pages/man1/mkfifo.1.html))
- aus einem C-Programm mit der Bibliotheksfunktion `mkfifo()` ([mkfifo(3)](https://man7.org/linux/man-pages/man3/mkfifo.3.html))

Sie bleibt (auch über Reboots hinweg) erhalten, bis sie explizit mit `rm` oder `unlink()` aus dem Dateisystem gelöscht wird.

Auf eine FIFO können _alle Prozesse_ entsprechend der gesetzten Dateizugriffsrechte zugreifen. Sie wird wie eine normale Datei geöffnet und geschlossen.

对同一个 文件 可以同时开启多个 writer 和 reader 

![](images/Pasted%20image%2020241210004019.png)

