
In Unix-Systemen besitzt jeder Prozess drei Standard-Streams zur Ein- und Ausgabe, die automatisch geöffnet und an das kontrollierende Terminal gebunden werden. Sie sind mit den _file descriptors_ (Dateinummern) 0-2 nummeriert:

| Name       | fd (_file descriptor_) | Verwendung                          |
| ---------- | ---------------------- | ----------------------------------- |
| **stdin**  | 0                      | Standard-Eingabe (Tastatur)         |
| **stdout** | 1                      | Standard-Ausgabe (Terminal)         |
| **stderr** | 2                      | Standard-Fehlermeldungen (Terminal) |

# 1 Umleitung 


| Operator | Effekt                                                     | Beispiel                     |
| -------- | ---------------------------------------------------------- | ---------------------------- |
| `<`      | stdin aus Datei lesen                                      | `wc < foo`                   |
| `>`      | stdout in Datei schreiben, bisherigen Inhalt überschreiben | `echo "hallo" > hello`       |
| `>>`     | stdout an Datei anhängen                                   | `echo "Welt" >> hello`       |
| `2>`     | stderr in Datei schreiben                                  | `wc /etc/* 2> /dev/null`     |
| `&>`     | stdin und stderr zusammen in eine Datei schreiben          | `grep user /etc/* &> result` |

## 1.1 例子 
ls > listing ; wc -l < listing

Antwort
Zuerst wird die Liste der Dateien des aktuellen Verzeichnisses in die Datei `listing` geschrieben, eine Datei pro Zeile. 
Danach wird die Anzahl der Zeilen in dieser Datei ausgegeben, also die Anzahl der Dateien im aktuellen Verzeichnis.

