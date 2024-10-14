> This is **not POSIX-compliant** and works in certain shell implementations like Bash 4+ but may not work in other shells.


Variables:
- `ghost`: This variable is **unset** (not defined).
- `file`: A path to a file `/home/user/unix/main/main.c` is assigned to this variable.
- datei=" / home/me/test/hallo. welt. c"
```
$ unset ghost
$ file=/home/user/unix/main/main.c
$ datei=" / home/me/test/hallo. welt. c"
```




1 Default Value:
- `${ghost:-hello}`: This checks if the variable `ghost` is defined. Since it's **unset**, the default value `hello` is printed.
- `${file:-hello}`: Since `file` is defined and not empty, it prints the value of `file`, which is `/home/user/unix/main/main.c`.

```sh
# Default-Wert, wenn Variable nicht definiert oder leer
$ echo ${ghost:-hello}
hello
$ echo ${file:-hello}
/home/user/unix/main/main.c
```




2 String Length:
- `${#file}`: The **length** of the `file` string is **27** characters.

```
$ echo ${#file}  # String-Länge
27
```





3 String Manipulation (Suffix Removal):
- `${file%.c}.o`: This **removes the suffix** `.c` from `file` and appends `.o` instead, resulting in `/home/user/unix/main/main.o`.
```
$ echo ${file%.c}.o  # Suffix .c entfernen, dann neuen Suffix .o anhängen    
/home/user/unix/main/main.o
```

den Dateinamen ohne die Endung .c ( /home/me/test/hallo.welt ) erhält man mit
`${datei%.*}`

Das Verzeichnis ( /home/me/test ) erhält man mit
` ${datei%/*}`




4 Extracting Filename:
- `${file##*/}`: This removes the longest prefix that matches `*/`, which is everything up to and including the last `/`, leaving just the filename `main.c`.
```
$ echo ${file##*/}  # Längsten Präfix, der "*/" matcht, entfernen
main.c
```

die Dateiendung ohne Punkt (c ) erhält man mit
`${datei##*.}`

Den Dateinahmen ohne Verzeichnis ( hallo.welt.c ) erhält man mit
`${datei##*/}`




 5 Replace Substring
    - `${file/main/src}`: This replaces the **first occurrence** of the substring `main` with `src`. The result is `/home/user/unix/src/main.c`.

```
### Die folgenden Beispiele sind nicht POSIX-konform
$ echo ${file/main/src}  # erstes Vorkommen von "main" durch "src" ersetzen
/home/user/unix/src/main.c
```





6 Uppercase Transformation
    - `${file@U}`: This transforms the entire `file` string to **uppercase**. The result is `/HOME/USER/UNIX/MAIN/MAIN.C`.
```
$ echo ${file@U}  # alles in UPPERCASE transformieren
/HOME/USER/UNIX/MAIN/MAIN.C
```