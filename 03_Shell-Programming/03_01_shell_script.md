
# 1 Parameter Expansion


## 1.1 `$#`

`$#` enthält die Anzahl der Argumente


## 1.2 `$@`

`$@` refers to all of a shell script’s command-line arguments.
代表 所有 在 xx.sh 后 加入的 argument 的值 

we use the special variable `$@`, which means, ‘All of the command-line arguments to the shell script’. We also should put `$@` inside double-quotes to handle the case of arguments containing spaces (`"$@"` is special syntax and is equivalent to `"$1"` `"$2"` …).


example 1
```
$ nano sorted.sh

# Sort files by their length.
# Usage: bash sorted.sh one_or_more_filenames
wc -l "$@" | sort -n

```

```
bash sorted.sh *.pdb ../creatures/*.dat

9 methane.pdb
12 ethane.pdb
15 propane.pdb
20 cubane.pdb
21 pentane.pdb
30 octane.pdb
163 ../creatures/basilisk.dat
163 ../creatures/minotaur.dat
163 ../creatures/unicorn.dat
596 total
```

example2 
```
nano do-stats.sh

# Calculate stats for data files.
for datafile in "$@"
do
    echo $datafile
    bash goostats.sh $datafile stats-$datafile
done
```


等效于 
```
bash do-stats.sh NENE*A.txt NENE*B.txt

bash do-stats.sh NENE*A.txt NENE*B.txt | wc -l


# Calculate stats for Site A and Site B data files.
for datafile in NENE*A.txt NENE*B.txt
do
    echo $datafile
    bash goostats.sh $datafile stats-$datafile
done

```



## 1.3 `$0`

`$0` den Namen des gerade ausgeführten Programms.



## 1.4 `$1`, `$2`

`$1`, `$2`, etc., refer to the first command-line argument, the second command-line argument, etc.


# 2 Ausführung von Shell Scripts

三种方法区执行 shell scripts

## 2.1 执行scripts 的命令的过程: main shell process and Subpshell rocess


Dies startet einen neuen Shell-Prozess (**Subshell** genannt), der das Script ausführt. Nach Abarbeitung des Scripts wird die Subshell beendet und Sie befinden sich wieder in der ursprünglichen Shell.

## 2.2 `#!/bin/bash` 的意义

#!/bin/bash  去表明 mit welchem Programm es ausgeführt werden soll.
  
Aus Sicht der Shell ist die `#!`-Zeile (_shebang_ oder _sha-bang_ genannt) ein Kommentar. Das Unix-System erkennt aber daran, dass es sich um ein Script handelt, und startet das angegebene Programm `/bin/bash` zu dessen Interpretation


## 2.3 `bash script.sh`

这种方法下, script.sh 中不需要加入  #!/bin/bash  去表明 mit welchem Programm es ausgeführt werden soll.



## 2.4 `./list.sh`

Das Shell Script direkt ausführbar machen (wie das geht, sehen wir gleich), dann kann es wie ein normales Programm direkt ausgeführt werden.
''
Machen Sie jetzt die Datei ausführbar: `chmod +x list.sh`. Dann können Sie diese mit `./list.sh` ausführen.
- **`chmod`**: This command is used to change the permissions of a file or directory.
- **`+x`**: This adds execute permissions to the file. After running this, the file can be executed as a program or script.
`chmod +x` on a file (your script) only means, that you'll make it executable. Right click on your script and chose **Properties** -> **Permissions** -> **Allow executing file as program**, leaves you with the exact same result as the command in terminal.





> Zur Ausführung des Scripts müssen Sie `./list.sh` eingeben, nur `list.sh` schlägt fehl. Warum?
>      Das aktuelle Directory ist nicht im PATH, daher muss der Pfad zum Script explizit angegeben werden.



## 2.5 `. list.sh`

Das Script mit dem Builtin `source` in eine laufende Shell einlesen.
Das Script mit dem Builtin `.` in die laufende Shell einzulesen (dazu muss das Script nicht ausführbar sein): `. list.sh`

在使用了这个方法之后  working directory 会变为  `/`  这个路径: 
Jetzt ist `/` das `working directory` der Shell. Da `source` das Script direkt in der interaktiven Shell ausführt, bleiben Zustandsänderungen wie Verzeichniswechel erhalten.



![](images/Pasted%20image%2020241006114901.png)

