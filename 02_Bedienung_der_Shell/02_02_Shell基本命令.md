
https://swcarpentry.github.io/shell-novice/index.html


`$` 代表 prompt 

|                     |                                                                                                                                                                                                         |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `exit`              | Shell beenden                                                                                                                                                                                           |
| Ctrl+c              | Laufenden Befehl vorzeitig beenden<br>Ein im Vordergrund laufendes Programm abbrechen                                                                                                                   |
| Ctrl+d              | _end of file_: Ende der aktuellen Eingabedatei kennzeichnen bzw. Shell beenden<br><br>**Shell beenden:** `Ctrl+D` (sendet das EOF-Signal, das die Shell beendet, wenn keine Eingabe mehr erwartet wird) |
| `Ctrl+Z`            | **Ein im Vordergrund laufendes Programm temporär stoppen:** `Ctrl+Z` (sendet das SIGTSTP-Signal, um das Programm anzuhalten und in den Hintergrund zu verschieben)                                      |
| Ctrl+s              | Terminal-Ausgabe temporär anhalten  这个记住                                                                                                                                                                |
| Ctrl+q              | Angehaltene Terminal-Ausgabe fortsetzen 这个记住                                                                                                                                                            |
| `clear` oder Ctrl+L | Bildschirminhalt löschen                                                                                                                                                                                |
| ← und →             | Cursor zum Editieren in der aktuellen Befehlszeile bewegen                                                                                                                                              |
| ↑ und ↓             | Blättern in der Befehlshistorie                                                                                                                                                                         |
| `history`           | Kommandohistorie als nummerierte Liste anzeigen                                                                                                                                                         |
| `!42`               | Kommando Nummer 42 aus der history-Liste wiederholen                                                                                                                                                    |
| `!ca`               | Letztes Kommando, das mit `ca` beginnt, wiederholen  这个记住                                                                                                                                               |
| `reset`             | Terminal zurücksetzen, wenn die Ausgabe unleserlich ist                                                                                                                                                 |

# 1 缺省值 

- `*` matches zero or more characters in a filename, so `*.txt` matches all files ending in `.txt`.
- `?` matches any single character in a filename, so `?.txt` matches `a.txt` but not `any.txt`.
- `$([command]) 
    - ` grep "searching" $(find . -name "*.txt")
    - `$([command])` inserts a command’s output in place.

# 2 Operator 

- `>` overwrites the file.
- `>>` appends to the file.


# 3 Exit Status `$?`

Der Wert 0 bedeutet eine erfolgreiche (fehlerfreie) Ausführung.

Der exit status des letzten Kommandos wird von der Shell in der speziellen Variablen `$?` gespeichert und kann so direkt abgefragt werden.

Das Shell-Builtin `exit n` beendet die aktuelle Shell mit dem exit status `n`.


## 3.1 Verbundene Kommandos

- `command1 ; command2` führt immer beide Kommandos nacheinander aus
- `command1 && command2` führt `command2` nur aus, wenn `command1` erfolgreich war (logisches AND)
- `command1 || command2` führt `command2` nur aus, wenn `command1` _nicht_ erfolgreich war (logisches OR)


# 4 命令 

## 4.1 ls 

|       |                                                                                                                                                                                                                                                                                                                                   |
| ----- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| ls -F | The `ls -F` command in Unix/Linux is used to list files and directories, with special characters appended to each entry to indicate their type. The `-F` option adds the following symbols:<br><br>- `/` for directories<br>- `*` for executable files<br>- `@` for symbolic links<br>- `\|` for named pipes<br>- `=` for sockets |
| ls -s | will display the size of files and directories alongside the names, <br>while `ls -S` will sort the files and directories by size, as shown below:                                                                                                                                                                                |
| ls -R | The `-R` option to the `ls` command will list all nested subdirectories within a directory. Let’s use `ls -FR` to recursively list the new directory hierarchy we just created in the `project` directory:                                                                                                                        |
|       |                                                                                                                                                                                                                                                                                                                                   |
|       |                                                                                                                                                                                                                                                                                                                                   |

## 4.2 mkdir

|          |                                                              |
| -------- | ------------------------------------------------------------ |
| mkdir -p | mkdir -p ../project/data    就是说 ../projekt/ 原本不存在, 但是能一起创造出来 |
|          |                                                              |

## 4.3 cp


| cp -r |  连带着文件底下的内容 都复制到 xx 处  |
| ----- | ---------------------- |
|       |                        |
|       |                        |



## 4.4 rm


| rm -r | 连带着文件底下的内容 都s删除  |
| ----- | ---------------- |
|       |                  |
|       |                  |



## 4.5 wc

`wc` is the ‘word count’
it counts the number of lines, words, and characters in files (returning the values in that order from left to right).


| wc -l | the number of lines in file content per file |
| ----- | -------------------------------------------- |
| -m    | the number of characters                     |
| -w    | the number of words                          |


## 4.6 sort 


|     | 默认情况下 是 the sort is alphanumerical. |
| --- | ----------------------------------- |
| -n  | the number of characters            |
|     |                                     |


# 5 cut

`cut -d , -f 2 animals.csv`

对每一行进行剪裁 

The `cut` command is used to remove or ‘cut out’ certain sections of each line in the file, and `cut` expects the lines to be separated into columns by a Tab character. 

A character used in this way is called a **delimiter**. In the example above we use the `-d` option to specify the comma as our delimiter character. We have also used the `-f` option to specify that we want to extract the second field (column).


# 6 Saerch

## 6.1 grep

grep "of" haiku.txt
grep "searching" $(find . -name "*.txt")
`$([command])` inserts a command’s output in place.

|     |                                                                                                                          |
| --- | ------------------------------------------------------------------------------------------------------------------------ |
| -w  | word boundaries<br><br>grep -w The haiku.txt<br>**-w 或 --word-regexp** : 只显示全字符合的列。<br>                                  |
| -n  | grep -n "it" haiku.txt<br><br><br>5:With searching comes loss<br>9:Yesterday it worked<br>10:Today it is not working<br> |
| -v  | invert our search                                                                                                        |
| -E  | Regular expressions<br><br>grep -E "^.o" haiku.txt                                                                       |


## 6.2 find 
找文件的名字 和文件夹的名字 

find’s output is the names of every file and directory under the current working directory

find . -type d
find’s output is the names of every file and directory under the current working directory

the . on its own means the current working directory
find’s output is the names of every file and directory under the current working directory


|       |                                                                                          |
| ----- | ---------------------------------------------------------------------------------------- |
| -type | -type d<br>‘things that are directories’.<br><br>-type f<br>‘things that are file ’.<br> |
| -name | find . -name *.txt                                                                       |
|       |                                                                                          |




