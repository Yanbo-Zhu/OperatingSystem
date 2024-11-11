
# 1 Inode

Ein **inode** (von _index node_) ist eine spezielle Datenstruktur im Dateisystem, die alle Metadaten zu einer Datei speichert. Zu jeder Datei und jedem Verzeichnis gehört genau ein inode, das durch eine im jeweiligen Dateisystem eindeutige **inode-Nummer** identifiziert wird.

每个文件和文件夹 都有一个对应的 inode , 里面包含了 meatadaten

Das inode enthält _nicht_ den Namen der Datei. Die Metadaten umfassen unter anderem
- Geräte-Nummer des Dateisystems
- inode-Nummer
- _link count_ (siehe unten)
- UID und GID des Eigentümers
- Dateityp und Zugriffsrechte
- Größe der Datei
- Zeitstempel
    

Sie können mit dem System Call `stat()` und dem Shell-Kommando `stat` abgefragt werden.


# 2 Verzeichniseinträge, Links

> Das geliche Datei auf meheren Verschnisse speichern

Verzeichniseintrag 就是 link, 关联 Dateinamen mit einer inode-Nummer
Ein Verzeichniseintrag verknüpft einen Dateinamen mit einer inode-Nummer im selben Dateisystem, zeigt also auf eine dort gespeicherte Datei. Diesen Eintrag nennt man auch einen Link zu dieser Datei.


对于同一个 datei 可以存在多个Link, link 可以存在在多个位置 
Es kann _mehrere Links_ auf dieselbe Datei geben, also mehrere Verzeichniseinträge mit derselben inode-Nummer. Diese Links können auch in verschiedenen Verzeichnissen existieren, die aber alle zum selben Dateisystem gehören müssen.


产生新的link  : ln 命令 
Mit dem Kommando `ln` (_link_) oder dem entsprechenden System Call `link()` erstellt man einen zusätzlichen Link auf eine bestehenden Datei:
    `ln foo bar` erstellt einen link namens `bar` zur bestehenden Datei `foo`.


Hard Link 就是
所有的 hard link 有同等的权利 
Alle Links zur selben Datei sind _völlig gleichberechtigt_, es gibt keinen Unterschied zwischen "Originaldatei" und "weiteren Links". Daher nennt man diese Art von Links auch **hard links** (im Unterschied zu den weiter unten behandelten symbolischen Links).
Hard links auf Verzeichnisse sind nicht zugelassen, weil dadurch Zyklen im Dateibaum entstehen können.


Hard line 一些特质 
>Bei mehreren Hard Links zu einer Datei kann man nicht erkennen, welcher zuerst da war.
>Hard Links können **nicht** kaskadiert werden: ein Link kann auf einen anderen Link zeigen.


# 3 Unlink

要想彻底的删除  文件 (也就是他的 Verzeichniseintrags 也被删除了), 需要将他的 hard links 先删除了才行
Beim Löschen eines Verzeichniseintrags darf die referenzierte Datei (inode) nur dann vom Dateisystem gelöscht werden, wenn es keine weiteren _hard links_ darauf gibt. Sonst wäre das Dateisystem inkonsistent.


Link count
Dazu speichert jedes inode einen **link count**, der die Anzahl der aktuell auf die Datei zeigenden Links enthält. Eine neu angelegte Datei hat einen _link count_ von 1. Für jeden weiteren _hard link_ darauf wird der _link count_ inkrementiert. Der Output von `ls -l` enthält den _link count_ zu jeder Datei.


其实文件从来不是被直接删除的, 当 link count = 0 , datei和他的node 就会被自动删除
Es gibt in Unix keine Möglichkeit zum direkten Löschen einer Datei, sondern nur den System Call `unlink()`. Dieser entfernt den angegebenen Verzeichniseintrag und dekrementiert den link count des referenzierten inode. Wenn der _link count_ dann 0 ist, werden inode und Datei automatisch gelöscht.

命令: rm 
Das Kommando `rm` verwendet intern ebenfalls `unlink()`. Es löscht also nicht die Datei, sondern den angegebenen Link.



# 4 Symbolische Links

> Jeder Symlink hat eine eigene eindeutige Inode-Nummer
> Symbolische Links auf Verzeichnisse sind zulässig.

和 hard link 不同 

当你打开文件的时候, 会自动将daetinhalt 填充到 symlink 里面,   就是说一个打开symlink 不是直接打开源文件, 而是将 symlink 的内容拷贝到另一个地方了
Ein **Symbolischer Link** (_symbolic link_, kurz _symlink_) ist eine spezielle Datei, die den absoluten oder relativen Pfad zu einer anderen Datei als Text enthält. Dieses _target_ kann überall im Dateibaum (also in jedem eingehängten Dateisystem) liegen.


Symbolische Links werden _zur Laufzeit_ aufgelöst: wenn ein Pfad geöffnet werden soll, der einen Symlink enthält, wird er durch dessen Inhalt ersetzt (der auch wieder ein Symlink sein kann). Dies ergeben sich andere Eigenschaften als bei _hard links_:

- Beim Löschen/Verschieben einer Datei kann nicht geprüft werden, ob Symlinks darauf existieren. Symlinks können daher _ungültig_ sein (nicht existentes _target_, Laufzeitfehler: _no such file or directory_).
    
- Der Zugriff über einen Symlink ist etwas langsamer als über den "echten" Dateipfad bzw. einen _hard link_, weil erst die Symlink-Datei geöffnet und gelesen werden muss. Durch Caching ist der Performanceverlust aber auf heutigen Systemen sehr gering.
    
- Symlinks auf Verzeichnisse sind zulässig (und werden oft verwendet).
    

ln -s 产生 软链接 
Ein Symlink wird mit dem Befehl `ln -s` oder dem System Call `symlink()` erzeugt. Dabei wird nicht geprüft, ob das _target_ existiert. Im Output von `ls -l` sind Symlinks am Dateityp `l` erkennbar.

# 5 `ls -il`

- Zeigt ebenfalls eine lange Listenansicht, enthält aber zusätzlich die **Inode-Nummer** jeder Datei.
- Die Inode-Nummer ist eine eindeutige Kennung, die das Dateisystem jeder Datei und jedem Verzeichnis zuweist.

```
1234567 -rw-r--r-- 1 user group 1024 Nov  9 12:00 example.txt
```
# 6 Unterschied zw symbolischen links und hard links erklären?


1 创造硬链接
![](images/Pasted%20image%2020241111201327.png)



![](images/Pasted%20image%2020241111201341.png)

2097160 数字是一样的 
2  是 关联这个文件的硬链接的数量 

---

ln -s 创造软连接 

![](images/Pasted%20image%2020241111201517.png)

最后一个是 symbolische Link 

![](images/Pasted%20image%2020241111201535.png)


---

insgesamt 8  是什么意思 

8 =   diese Dateien, wie viel Speicherplatz verbrauchen 

Anzahl von benotige Blöcke der Bedarf    =    20 byte  groß ist dann verbraucht die 4 KB 

![](images/Pasted%20image%2020241111201809.png)


# 7 源文件删除后, 之前创造的链接还能用吗 

![](images/Pasted%20image%2020241111202153.png)


hello4 是 hello2 的软连接 , 通过 ln hello2 hello4 产生
hello3 是 hello2 的硬链接   通过 ln -s hello2 hello3 产生

删除 hello2 后 
hello3 仍存在, 可以打开 可以使用 

hello4 仍存在, 但是无法使用了
also wenn man hello4 öffnen, will hello4  einen  fehler werfen, (broken symlink) . Symlinks werfen eben keine Fehler, wenn das Storage-Backend weg ist.
man kann usern oder prozessen vorgaukeln, auf eine Datei zuzugreifen, wenn man z.B. eine Config-Datei durch einen Softlink ersetzt 😁
![](images/Pasted%20image%2020241111202359.png)







# 8 有些文件不能生成 hart link 

1
![](images/Pasted%20image%2020241111202741.png)


2 
Können Hard-Links auf Verzeichnisse zeigen?
hard linke not allowed for directory 
directory 不能创造硬链接 

![](images/Pasted%20image%2020241111203347.png)
