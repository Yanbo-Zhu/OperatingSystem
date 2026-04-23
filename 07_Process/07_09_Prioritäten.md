

Die Scheduling-Entscheidungen des Kernels können über Prozess-Prioritäten beeinflusst werden.


 höherer Priorität 的优势
Die genaue Auswirkungen der Prioritäten hängt vom konkreten Scheduling-Algorithmus ab. Prozesse höherer Priorität können z.B.
- schneller eine CPU zugeteilt bekommen (kürzere Wartezeit)
- nicht oder seltener unterbrochen werden
- insgesamt einen höheren Anteil an der CPU-Zeit erhalten

In Unix-Systemen gibt es dafür zwei prinzipielle Mechanismen, die nebeneinander existieren:

# 1 Scheduling Priority (for Realtime-Prozesse)

Realtime-Prozesse haben eine ganzzahlige statische **scheduling priority**. Prozesse mit numerisch höherem Prioritätswert werden vor Prozessen mit niedrigerem Prioritätswert zugeteilt.

POSIX schreibt mindestens 32 Prioritätsstufen vor. In Linux kann die _scheduling priority_ zwischen 1 (niedrigste) und 99 (höchste) liegen.

In Linux können Sie mit dem Kommando `chrt` die Echtzeit-Scheduling-Attribute eines Prozesses anzeigen oder (als `root`) ändern.


## 1.1 Nice Value ( for Normale (nicht-Echtzeit) Prozesse) 

Normale (nicht-Echtzeit) Prozesse haben immer die _scheduling priority_ 0, d.h. ==jeder Echtzeit-Prozess hat Vorrang vor jedem normalen Prozess==. 
Um innerhalb der normalen Prozesse unterschiedliche Prioritäten setzen zu können, hat jeder normale Prozess einen zweiten Prioritätswert.

Dieser sogenannte **nice value** liegt zwischen **-20 (höchste Priorität)** und **+19 (niedrigste Priorität)**. Der Default-Wert ist 0.

==Ein höherer _nice value_ bedeutet _niedrigere_ Priorität, also umgekehrt wie bei der _scheduling priority_.   ("_a process with high nice value is nicer to others_"  总把机会让给别人)==


Es gibt verschiedene Schnittstellen zum Zugriff auf den _nice value_:
- Mit den System Calls `getpriority()` und `setpriority()` kann ein Prozess seinen _nice value_ lesen bzw. setzen.
- Das Kommando `nice` startet ein Programm mit einem gegebenen _nice value_.
- Das Kommando `renice` ermöglicht es, den _nice value_ eines laufenden Prozesses zu ändern.
- `ps -l` zeigt den _nice value_ in der Spalte "NI" an.

Prozess 有negativem _nice value_ , 代表这个process 需要  besondere Privilegien
Um einen Prozess mit negativem _nice value_ zu starten, besondere Privilegien (z.B. root-Rechte oder in Linux die _capability_ CAP_SYS_NICE) benötigt.


Eine unprivilegierter Prozess kann seinen eigenen _nice value_ im Normalfall nur erhöhen (d.h. seine Priorität senken),  _nice value_ 只能升高 不能下降 
Aus Sicherheitsgründen kann ein unprivilegierter Prozess seinen eigenen _nice value_ im Normalfall nur erhöhen (d.h. seine Priorität senken), nicht aber reduzieren. Auch eine Rückkehr zum Ausgangswert ist nicht mehr möglich, nachdem der _nice value_ einmal erhöht wurde.

_nice value_  不是 绝对的Priorität
Im Unterschied zur _scheduling priority_ drückt der _nice value_ keine absolute Priorität, sondern eher eine _Präferenz_ aus: Prozesse mit niedrigerem _nice value_ sollen einen höheren Anteil CPU-Zeit erhalten. Der genaue Effekt hängt vom verwendeten Scheduler-Algorithmus ab.
