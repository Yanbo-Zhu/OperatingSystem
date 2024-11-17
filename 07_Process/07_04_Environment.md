

Jeder Prozess in Unix hat ein **Environment**. Dies besteht aus einer Menge von Variablen.  
Die Werte sind jeweils Strings.  
Die Variablennamen bestehen aus Großbuchstaben, Ziffern, und "_" ; das erste Zeichen darf keine Ziffer sein.

In C-Programmen ist das gesamte Environment eines Prozesses als _globale Variable_ zugänglich:

```c
extern char **environ;
```


Diese Variable ist ein unsortiertes, NULL-terminiertes Array von Zeigern auf Strings der Form "VARIABLE=WERT". Um direkt darüber auf eine bestimmte Environment-Variable zuzugreifen, muss man erst nach dem Namen suchen und dann aus dem String "VARIABLE=WERT" den Wert extrahieren.

Weil dies sehr umständlich wäre, definiert der POSIX-Standard _Zugriffsfunktionen_, die in der Standard-Library implementiert werden:

|   |   |
|---|---|
|`getenv(name)`|gibt den Wert der Variablen zurück (oder `NULL`, falls nicht definiert).|
|`setenv(name, value, 1)`|Weist der Variablen den Wert `value` zu|
|`setenv(name, value, 0)`|Zuweisung erfolgt nur, wenn die Variable noch keinen Wert hat|
|`unsetenv(name)`|Löscht die Variable aus dem Environment|

Bei `fork()` wird das Environment an den neuen Kindprozess vererbt; dieser erhält eine _identische Kopie_. Spätere Änderungen des Environments im Eltern- oder Kindprozess sind unabhängig voneinander.

Über Environment-Variablen werden typischerweise Informationen und Einstellungen verteilt, die für eine Gruppe von Prozessen relevant sind, z.B. alle Prozesse einer Benutzer-Session. Beispiele sind:

- `EDITOR`: vom Benutzer festgelegter Default-Texteditor
    
- `HOME`: Home-Directory des aktuellen Benutzers
    
- `PATH`: Suchpfad für ausführbare Programme
    
- `TERM`: Typ des zugeordneten Terminals
    
- `USER`: aktueller User

Sehen Sie sich mit das aktuelle Environment Ihrer Shell an: `printenv | sort`. Recherchieren Sie exemplarisch, was einige der Variablen bedeuten.


