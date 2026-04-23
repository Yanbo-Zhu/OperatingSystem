
C ist eine kompilierte Sprache. Das bedeutet, dass Ihr geschriebener C-Code nicht direkt ausgeführt werden kann, sondern in eine ausführbare Maschinencode-Datei übersetzt werden muss. Dieser Prozess wird Kompilieren oder im Englischen building genannt. 

Für die Aufgaben verwenden wir CMake1 als Build-System. CMake erspart uns sehr viel kleinteilige Konfigurationsarbeit, indem wir für unser gesamtes Projekt nur eine globale Konfigurations-Datei, die CMakeLists.txt, erstellen müssen.


# 1 hello_world

```
-- dir
----build
----CMakeLists.txt
----hello_world.c
```


hello_world.c
```c
#include <stdio.h>
#include <stdlib.h>
 
int main(int argc, char *argv[])
{
    printf("Hello World!");
	//system("pause");
 
    return 0;
}
```


CMakeLists.txt
```
project(RN-Praxis) # Der Name unseres Projekts
cmake_minimum_required(VERSION 3.5) # Minimale CMake-Version 3.5
set(CMAKE_C_STANDARD 11) # Wir verwenden C-Version C11

# Diese Zeile ist die Anweisung, dass wir eine ausführbare Datei erstellen wollen
add_executable(hello_world hello_world.c)


# Wenn Sie mehrere Code-Dateien haben, werden diese Leerzeichen getrennt angehängt:
# add_executable(executable first.c second.c third.c)

# Diese Zeilen sind für das Erstellen der Abgabedatei relevant
set(CPACK_SOURCE_GENERATOR "TGZ") # Abgabe soll als .tar.gz erstellt werden

# Die fertige Abgabe enthält nur den Quellcode und nicht ihr Build-Verzeichnis
set(CPACK_SOURCE_IGNORE_FILES ${CMAKE_BINARY_DIR} /\\..*$)
set(CPACK_VERBATIM_VARIABLES YES) # Variablen sollen nicht optimiert werden
include(CPack) # Wir nutzen CPack um das Archiv zu erstellen~
```


# 2 Code kompilieren

Mit dieser CMakeLists.txt Datei können Sie nun erst ihr build-Verzeichnis, und dann ihre ausführbare Datei wie folgt erstellen.
```shell
cmake -B build -D CMAKE_BUILD_TYPE=Debug # Erstellt das Build-Verzeichnis
make -C build # Kompiliert in ’build’ wie per add_executable spezifiziert
# Alternativ kürzer: ’cmake -B build -D CMAKE_BUILD_TYPE=Debug && make -C build’
```


# 3 Code ausführen

make -C build 后 
Datei hello_world in Build Folder 被生成

执行它 ./build/hello_world


# 4 Abgabedatei erstellen

Um Ihre fertige Abgabe einzureichen, müssen Sie die Abgabedatei mit folgenden Befehl erstellen:
`make -C build package_source # Erstellt ihre Abgabedatei im ’build’ Ordner`


