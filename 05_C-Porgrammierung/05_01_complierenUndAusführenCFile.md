
借助于 GCC 

Installieren Sie zuerst den C-Compiler GCC sowie die benötigten Bibliotheken und Header-Files. In Ubuntu gibt es dafür das Paket `build-essential`:

$ sudo apt install build-essential


# 2 hello.c 为例子 

Legen Sie mit einem Texteditor die Quellcode-Datei hello.c mit folgendem Inhalt an:

```c
#include <stdio.h> 

int main() { 
  puts("Hello, World!\n"); 
  return 0; 
}
```

1	Mit dem header file stdio.h werden Deklarationen für I/O-Funktionen wie puts() importiert.
2	Beim Start eines C-Programms wird die Funktion main() aufgerufen. Mit dem Rücksprung aus main() endet das Programm.
3	puts() gibt einen String auf stdout aus. \n kennzeichnet das Zeilenende (newline). Statt puts() wird häufig die Funktion printf() verwendet, die formatierte Ausgaben ermöglicht.
4	Der Rückgabewert von main() ist der exit status des Programms: 0 steht unter Unix (POSIX) für erfolgreiche Beendigung, andere Werte für einen Fehler. Diese Zeile kann weggelassen werden: Der Compiler fügt am Ende von main() automatisch Code ein, der 0 zurückgibt. Nur im Fehlerfall muss der Rückgabewert explizit angegeben werden, z.B. return(1). 

kann man die Datei mit GCC compilieren:
`$ gcc -o hello hello.c`
Die Option -o hello gibt den Namen des ausführbaren Programms an (der default-Wert ist a.out).
得到 hello 这个文件 


Jetzt können Sie das Programm ausführen:
`$ ./hello `
Hello, World!
