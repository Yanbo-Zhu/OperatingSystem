


# 1 Eigentümer und Gruppe einer Datei (`chown` and `chgrp` ) 

Jede Datei gehört zu genau einem Benutzer und genau einer Gruppe

- Mit dem Kommando `chown` (_change owner_) kann root den Eigentümer einer Datei beliebig ändern.
- Mit `chgrp` (_change group_) kann der _owner_ einer Datei diese einer anderen Gruppe zuordnen, in der er selbst Mitglied ist. root kann jeder Datei eine beliebige Gruppe zuweisen.


# 2 Zugriffsrechte chmod


Maske  

```
Benutzerkategorie: ---uuugggooo
Zugriffsrecht:     sgtrwxrwxrwx
Wert:              000111101001
```

--- -> sgt -> 000

uuu -> rwx -> 111  -> 7 代表这个文件 所属的 user 对这个file 有 **r**ead, **w**rite, e**x**ecute. 的权利 

ggg ->  rwx  -> 101 -> 5 代表这个文件 所属的group的其他的user   对这个file 有 **r**ead,  e**x**ecute. 的权利 , 没有 **w**rite 的权利 
ooo -> rwx -> 001  -> 1  代表 alle anderen Benutzer 对这个fie 只有 执行的去阿里 


|Zugriffsrecht|Bedeutung für reguläre Dateien|Bedeutung für Verzeichnisse|
|---|---|---|
|r|Datei lesen|Verzeichnis-Einträge lesen|
|w|Datei schreiben|Verzeichnis schreiben: Dateien darin anlegen, löschen, umbenennen|
|x|Datei als Programm ausführen|In das Verzeichnis wechseln|



## 2.1 例子 


Der Output von `ls -l` zeigt die Zugriffsrechte in Form der unteren 9 Bits sowie Eigentümer und Gruppe einer Datei an:
`-rwxr-x--- 1 alice  students     4 May 19 09:59  test`
Dies entspricht oktal der Maske `0750`.
- - : 代表是 file, 不是 directory 
- user: alice darf die Datei lesen, schreiben und ausführen
- group: Mitglieder der Gruppe students dürfen die Datei lesen und ausführen
- others: alle anderen Benutzer haben keine Zugriffsrechte




# 3 setuid, setgid, sticky


Maske  
```
Benutzerkategorie: ---uuugggooo
Zugriffsrecht:     sgtrwxrwxrwx
Wert:              000111101001
```


--- -> sgt -> 000

s: setuid   s对应得值是1, 不是0的话, 代表  这个文件 可以使用 setuid 这个功能
g: setgid
t: stickty


Die obersten (linken) 3 Bits der Zugriffsrechte-Maske werden `setuid`, `setgid` und `sticky` genannt und haben eine besondere Bedeutung.  Auf setuid und setgid gehen wir unten genauer ein.


## 3.1 sticky 

Das `sticky`-Bit ist heute nur noch für Verzeichnisse relevant: ==In einem Verzeichnis mit diesem Bit dürfen unprivilegierte Benutzer nur Dateien löschen oder umbenennen, wenn ihnen die Datei oder das Verzeichnis gehört.== 
Es wird oft für Verzeichnisse verwendet, in denen mehrere Benutzer Daten ablegen können.



## 3.2 setuid

Das **setuid**-Bit in den Zugriffsrechten kann nur für ausführbare Dateien gesetzt werden und hat eine besondere Bedeutung: ==Wenn diese Datei ausgeführt wird, _erhält der ausführende Prozess die User-ID der Datei_.==   那个user在操作 datei, 他就是 die User-ID der Datei. 
process 获得 这个UserID, 借用这个身份, 去操作关于这个文件的一些东西 

Ob ein Benutzer die Datei ausführen darf, ergibt sich aus den normalen e**x**ecute-Rechten. 
 这个user 是否能执行这个Datei, 取决于 这个 file 自己定义的 Zugriffsrechte


Dieser Mechanismus ist in Unix die einzige Möglichkeit für unprivilegierte Benutzer, ein Programm unter einer fremden User-ID auszuführen. Er wird insbesondere verwendet, um normalen Benutzern in einem eingeschränkten Rahmen privilegierte Aktionen zu erlauben.

sudo和su命令就是  借用了 setuid 
Die Programme `sudo` und `su` sind _setuid root_, laufen also unter root. Sie enthalten aber eigene Mechanismen zur Zugriffskontrolle über `/etc/sudoers` bzw. die Eingabe des root-Passworts, damit sich Benutzer nicht beliebige Privilegien verschaffen können.



## 3.3 setgid 

Das **setgid**-Bit ermöglicht es analog, ein Programm unter der Gruppen-ID der Datei auszuführen. Es wird in der Praxis kaum verwendet.


process 获得 这个Gruppen-ID der Date, 借用这个身份, 去操作一些东西 


# 4 Access Control Lists

> 除了 _user/group/others_ 额外的Zugriffsrechten如何添加 

Beispielsweise An einer Datei kann optional eine ACL hinterlegt werden, die zusätzliche Zugriffsrechte für weitere Benutzer und/oder Gruppen enthält — jeweils wieder nach r/w/x unterschieden.


ACLs können mit dem Kommando `getfacl` angezeigt und mit `setfacl` geändert werden. 

`ls` kann den Inhalt von ACLs nicht direkt anzeigen. Im Output von `ls -l` wird aber die Existenz einer ACL mit einem `+` hinter der Zugriffsrechte-Maske markiert.

In Ubuntu finden Sie im Verzeichnis `/dev` einige Dateien mit ACLs. Sehen Sie sich die ACLs dazu mit `getfacl` an.

