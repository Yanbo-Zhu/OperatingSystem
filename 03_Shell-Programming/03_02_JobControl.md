
# 1 foreground and background

forground: 
只有当前一个command 被执行结束, 出现新的 $ 标志, 后一个command才会被出现 
Normalerweise laufen gestartete Shell-Kommandos als Job im **Vordergrund (_foreground_)**: Solange das Kommando (genauer: der entsprechende Prozess) ausgeführt wird, sehen Sie im Terminal nur dessen Output. Die Shell wartet, bis der Prozess beendet ist. Dann gibt sie das Prompt `$` aus und ist bereit für den nächsten Befehl. Mit Ctrl+c können Sie das laufende Kommando abbrechen.


Background: (with `command &`)
前一个process 被启动后, 后一个command 就已经准备好可以执行了, 马上被执行了
Man kann ein Shell-Kommando auch im **Hintergrund (_background_)** starten, in dem man ein `&` anhängt:  
`command &`. Der entsprechende Prozess wird gestartet und die Shell ist sofort für das nächste Kommando bereit. Das ist nützlich für länger laufende Kommandos, die keine interaktiven Eingaben des Benutzers erfordern, z.B. Archivierung, Backup, oder langwierige Suchen.

```
Backup des eigenen home directory als tar-Archiv mit Log-Datei:

tar -cvzf backup.tar.gz ~ &> backup.log &
```


# 2 和 Vorder- und Hintergrund 有关系的命令 


- `Ctrl+z` stoppt den im Vordergrund laufenden Prozess temporär (_suspend_).
    - A "background job" is just one that is not interacting with the user -- it doesn't control the tty and it just does its thing (generally silently). A foreground job is the reverse, it holds control of the tty to interact with the user.
    - Control-Z suspends the most recent foreground process (the last process to interact with the tty) (unless that process takes steps to ignore suspension, like shells normally do). This will generally bring you back to your shell, from which you can generally enter the command `bg` to move the just-suspended process to the background (letting it continue to run) or `fg` to bring it back to the foreground.
- `jobs` listet die aktuellen im _background_ laufenden oder gestoppten Jobs. Die jobs sind in der Liste durchnummeriert und können über diese Nummern ausgewählt werden.
- `fg` (_foreground_) setzt einen gestoppten oder im Hintergrund laufenden Prozess im Vordergrund fort. 继续执行 某个之前暂停的 Hintergrund process , 使得这个process 在 Foreground被继续执行 
- `bg` (_background_) setzt einen gestoppten Prozess im Hintergrund fort.  继续执行某个之前暂停的 background process 
- `kill %1` beendet Job Nummer 1 über ein Signal.


![](images/Pasted%20image%2020241006121545.png)



