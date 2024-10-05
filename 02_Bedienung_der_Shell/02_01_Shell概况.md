
Eine Shell kann eingegebene Kommandos auf zwei Arten erhalten:
- direkt vom Benutzer über ein textbasiertes Terminal (**interaktive Shell**)
- aus einer Programmdatei mit einer Folge von Shell-Befehlen, einem sog. **Shell Script**.


verschiedene Implementierungen der Unix-Shell
- _Schlanke Shells_ (z.B. `ash`, `dash`) streben eine speicher- und laufzeiteffiziente Implementierung des POSIX-Standards an, haben aber kaum zusätzliche Funktionen. Sie eignen sich besonders für die nicht-interaktive Benutzung in Shell Scripts sowie für ressourcenarme Systeme.
- _Komfortable Shells_ (z.B. `bash`, `fish`, `tcsh`, `zsh`) erweitern den POSIX-Funktionsumfang um zusätzliche Sprachkonstrukte (z.B. Array-Variablen) und Komfortfunktionen, z.B.
- _History-Mechanismus_: Blättern, Durchsuchen und Editieren früher eingegebener Kommandos
- _Tab completion_: Automatische Vervollständigung teilweise eingegebener Kommandos mit Tab





