
A. Analysieren Sie 'ls -l' mittels strace(1). Analysieren Sie die Ausgabe, und beantworten Sie die folgenden Fragen:

welche Systemrufe dienen dazu, Informationen über das aktuelle Verzeichnis zu ermitteln?
welche Systemrufe dienen dazu, die Ausgabe zu erzeugen?
Geben Sie jeweils die konkreten Systemrufe mit Parametern an, eine Erklärung, was diese im Allgemeinen machen, und wie die Parameter konkret zu interpretieren sind.

B. Führen Sie auf Ihrem Linux-System lsmod aus. Wählen Sie zufällig 3 Module. Ermitteln Sie, welchem Zweck das Modul dient, und geben Sie einen Link zum Quelltext dieses Moduls an. Achten Sie dabei auf die korrekte Version des Quelltexts.



A
strace 的作用是 wirklich welcher konkrete System Aufruf jetzt ausgeführt wurde
A 的目标是 angukcen, also wo wird jetzt im Fall von ls mir wirklich der Verzeichnis Inhalt ausgelesen und kleiner Tipp

strace 的目的是 Ausgaben zu lesen


![](images/Pasted%20image%2020250106204814.png)


![](images/Pasted%20image%2020250106205216.png)






B


die korrekte Version des Quelltexts.

![](images/Pasted%20image%2020250106203657.png)

跳到 alter version
指示处 ordner von module 就足够了 
![](images/Pasted%20image%2020250106203726.png)

![](images/Pasted%20image%2020250106204301.png)