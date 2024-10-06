

**process substitution**, which allows you to treat the output of a command as if it were a file. This is useful when you want to use the output of one command in place of a file input for another.
Process substitution (kein POSIX-Feature, aber in Bash implementiert) ermöglicht es, den Standard-Output eines Kommandos als temporäre Datei an ein anderes Kommando zu übergeben, das einen Dateinamen als Argument erwartet.

---

example1: 
Sie möchten die Listings der Verzeichnisse /bin und /usr/bin mit diff vergleichen. 

Ohne process substitution müssen Sie temporäre Dateien verwenden und dann wieder löschen:
```
ls /bin > binlist
ls /usr/bin > usrbinlist
diff binlist usrbinlist
rm binlist usrbinlist
```

Mit process substitution geht das viel kürzer:
diff <(ls /bin) <(ls /usr/bin)



---
example2
Probieren Sie die Kommandos `cat <(ls)` und `echo <(ls)` aus. Was wird jeweils ausgegeben?

`cat <(ls)` gibt das Directory-Listing aus, `echo <(ls)` den Pfad der automatisch erzeugten temporären Datei, z.B `/dev/fd/63`. 这个Pfad里面包括了 因为 <(ls) 产生的 临时文件的名字 


`cat <(ls)`
- `<(ls)` is **process substitution**, which runs the `ls` command and gives its output through a temporary file-like object.
- `cat <(ls)` reads from that file-like object and displays the contents of it, which is the **output of `ls`** (i.e., a directory listing).

**Result**: You will see the output of the `ls` command, i.e., a list of files and directories in the current directory.

`echo <(ls)`
- In this case, `<(ls)` again runs the `ls` command, but `echo` doesn't read the contents of the output like `cat` does.
- Instead, `echo` just prints the **file descriptor** that the system creates for the command's output, typically something like `/dev/fd/63`.

**Result**: You will see the path of the temporary file (like `/dev/fd/63`) created by process substitution, but not the contents of the directory listing.


- **Process substitution** (e.g., `<(command)`) creates a temporary file or a file descriptor (like `/dev/fd/63`) that points to the output of `command` (`ls` in this case).
- `cat` reads the contents of this file descriptor and outputs it (the directory listing).
- `echo`, on the other hand, just prints the **file path** to the file descriptor but does not process its contents.

Thus:
- `cat <(ls)` gives the **directory listing** (contents of the command output).
- `echo <(ls)` shows the **path to the file descriptor** that represents the command's output.
