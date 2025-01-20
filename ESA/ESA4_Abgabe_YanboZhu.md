# 1 A:  Analysieren Sie 'ls -l' mittels strace(1)

Analysieren Sie 'ls -l' mittels strace(1)

Analysieren Sie die Ausgabe, und beantworten Sie die folgenden Fragen:
- welche Systemrufe dienen dazu, Informationen über das aktuelle Verzeichnis zu ermitteln?
- welche Systemrufe dienen dazu, die Ausgabe zu erzeugen?

Geben Sie jeweils die konkreten Systemrufe mit Parametern an, eine Erklärung, was diese im Allgemeinen machen, und wie die Parameter konkret zu interpretieren sind.

## 1.1 Zusammenfassung 

These system calls work together to ensure `ls -l` retrieves the necessary data and formats it for display in the terminal.

**Retrieve Information:**
- `openat` (Opens the directory).
- `getdents64` (Reads directory entries).
- `stat` or `newfstatat` (Gets metadata for files).

**Generate Output:**
- `write` (Writes the formatted file information to the terminal).
- `close` (Cleans up file descriptors).

Procedure 
- Der Befehl `ls -l` nutzt `openat`, um ein Verzeichnis zu öffnen.
- Der Befehl `ls -l` nutzt `getdents64`, um die Inhalte eines Verzeichnisses zu lesen.
- Danach ruft es zusätzliche Systemaufrufe wie `stat` oder `newfstatat` auf, um Details zu jeder Datei zu ermitteln (z. B. Berechtigungen, Größe).
    - Diese Metadaten (z. B. Typ, Berechtigungen, Größe, Eigentümer) werden dann für die Ausgabe formatiert und angezeigt.
- Schließlich wird die formatierte Ausgabe mit `write` auf dem Bildschirm angezeigt.
- Am End `close` ausführen, um alle relevante File Descriptors aufzuräumen




## 1.2 Systemrufe zur Ermittlung von Informationen über das aktuelle Verzeichnis

### 1.2.1 `openat`

- **Funktion:** Öffnet ein Verzeichnis oder eine Datei mit den angegebenen Flags. 
- **Parameter:**
    - `dirfd`: Ein Verzeichnisdateideskriptor 
        - `AT_FDCWD` für das aktuelle Verzeichnis
    - `pathname`: Der Pfad des Verzeichnisses (z. B. `"./"`).
    - `flags`: Zugriffsmodi (z. B. `O_RDONLY` für nur Lesen).


---

Beipsiel 1 
`openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3`

Dieser Aufruf wird von vielen Programmen verwendet, um auf den dynamischen Bibliothekscache zuzugreifen, bevor dynamische Bibliotheken geladen werden. Der Cache enthält eine optimierte Liste von Speicherorten für Bibliotheken und beschleunigt die Suche.

Parameter 
- **`AT_FDCWD`:**
    - Steht für "File Descriptor for Current Working Directory".
    - Dies gibt an, dass der Pfad relativ zum aktuellen Arbeitsverzeichnis interpretiert wird (oder absolut, wenn der Pfad mit `/` beginnt).
- **`"/etc/ld.so.cache"`:**
    - In diesem Fall wird die Datei `/etc/ld.so.cache` geöffnet.
    - Diese Datei enthält einen Cache für dynamische Bibliotheken, um deren Ladezeit zu optimieren.
- **`O_RDONLY`:**
    - Öffnet die Datei nur zum Lesen (Read-Only).
- **`O_CLOEXEC`:**
    - "Close-on-Exec": Der Dateideskriptor wird automatisch geschlossen, wenn ein neuer Prozess mit `exec` gestartet wird.
    - Dies verhindert, dass offene Deskriptoren versehentlich an Kindprozesse weitergegeben werden.

Rückgabewert 
- **`= 3`:**
    - Der Rückgabewert ist ein Dateideskriptor (in diesem Fall `3`), der die geöffnete Datei repräsentiert.
    - Dieser Wert wird für nachfolgende Operationen (z. B. Lesen mit `read`) verwendet.

---

Beispiel 2
`openat(AT_FDCWD, ".", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3`

- Öffnet das aktuelle Verzeichnis (`"."`) im Lese- und Verzeichnismodus.
- Parameter:
    - `AT_FDCWD`: Basierend auf dem aktuellen Arbeitsverzeichnis.
    - `"."`: Gibt das aktuelle Verzeichnis an.
    - Flags: `O_RDONLY`, `O_NONBLOCK`, `O_CLOEXEC`, `O_DIRECTORY`.



### 1.2.2 `getdents64`

- **Funktion:** Liest Einträge aus einem geöffneten Verzeichnis. `getdents64` ist ein zentraler Systemaufruf für das Verzeichnis-Listing und liefert die Basisinformationen über die enthaltenen Einträge.
- **Parameter:**
    - `fd`: Dateideskriptor des Verzeichnisses.
    - `dirp`: Ein Puffer für die Verzeichniseinträge.
    - `count`: Die maximale Anzahl von Bytes, die gelesen werden können.


---

Beispiel
`getdents64(3, 0x563c82e25930 /* 28 entries */, 32768) = 920`

`getdents64` liest Verzeichniseinträge aus und speichert sie in einem Puffer, der von der Adresse `0x563c82e25930` repräsentiert wird.
Diese Einträge enthalten Metadaten wie:
- **Inode-Nummer** (`d_ino`): Identifiziert die Datei im Dateisystem.
- **Name** (`d_name`): Der Name der Datei/des Verzeichnisses.
- **Typ** (`d_type`): Der Dateityp (z. B. reguläre Datei, Verzeichnis, Symbolischer Link).

Parameter
- **Dateideskriptor (FD):**
    - `3`: Der Deskriptor des geöffneten Verzeichnisses (zuvor mit `openat` geöffnet).
- **Buffer-Adresse:**
    - `0x563c82e25930`: Ein Zeiger auf den Speicherbereich, in den die Verzeichniseinträge geladen werden.
    - Dieser Speicherbereich enthält 28 Einträge (z. B. Dateinamen, Inode-Nummern, Dateitypen).
- **Buffer-Größe:**
    - `32768`: Maximale Größe des Puffers in Bytes, die für die Einträge bereitgestellt wird.

**Rückgabewert:**
- `920`: Die tatsächlich gelesenen Bytes, die den Einträgen im Verzeichnis entsprechen.

### 1.2.3 `newfstatat`

- **Funktion:** Der Systemaufruf `newfstatat` wird verwendet, um Metadaten (z. B. Dateityp, Größe, Berechtigungen) zu einer Datei oder einem Verzeichnis zu ermitteln. Schauen wir uns den Aufruf genauer an.  Der Aufruf ermittelt Attribute einer Datei oder eines Verzeichnisses, ohne es erneut öffnen zu müssen.
- **Parameter:**
    - `dirfd`: Verzeichnisdateideskriptor.
    - `pathname`: Der Pfad zur Datei (z. B. `"file1.txt"`).
    - `flags`: Zugriffsmodi (z. B. `AT_SYMLINK_NOFOLLOW`).
    - `mask`: Welche Informationen abgefragt werden sollen (z. B. `STATX_BASIC_STATS`).
    - `statxbuf`: Ein Puffer, in den die Metadaten geschrieben werden.


---

Beispiel 
`newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=18699, ...}, AT_EMPTY_PATH) = 0`

Der Aufruf `newfstatat` in diesem Beispiel wird verwendet, um Informationen zu einer Datei (z. B. `/etc/ld.so.cache`) zu ermitteln. Der Dateityp (`S_IFREG`) und die Berechtigungen (`0644`) sind entscheidend, um die Ausgabe von `ls -l` korrekt darzustellen.


Parameter 
- **Dateideskriptor (FD):**
    - `3`: Der Dateideskriptor, der mit einem vorherigen `open` oder `openat` Systemaufruf geöffnet wurde. Dieser Deskriptor verweist hier auf eine Datei (z. B. `/etc/ld.so.cache`).
- **Pfadname:**
    - `""`: Ein leerer String bedeutet, dass der Aufruf den Dateideskriptor (`3`) verwendet, um die Datei zu identifizieren.
    - Der `AT_EMPTY_PATH`-Flag erlaubt es, den Pfad leer zu lassen und sich nur auf den Dateideskriptor zu beziehen.
- **Rückgabewert (Stat-Struktur):**
    - `{st_mode=S_IFREG|0644, st_size=18699, ...}`: Eine Struktur, die Dateiinformationen enthält:
        - **`st_mode=S_IFREG|0644`**: Die Datei ist eine reguläre Datei mit Berechtigungen `0644` (lesbar und schreibbar vom Besitzer, lesbar für andere).
        - **`st_size=18699`**: Die Dateigröße beträgt 18.699 Bytes.
- **Flags:**
    - `AT_EMPTY_PATH`: Dieser Flag erlaubt die Verwendung eines Dateideskriptors anstelle eines vollständigen Pfadnamens. Dies ist hilfreich, wenn der Dateideskriptor bereits die Datei oder das Verzeichnis referenziert.


**Rückgabewert des Aufrufs:**
- `= 0`: Signalisiert, dass der Aufruf erfolgreich war.


## 1.3 Systemrufe zur Erzeugung der Ausgabe
### 1.3.1 `write`

- **Funktion:** Schreibt Daten in eine Datei oder einen Ausgabestrom (z. B. `stdout`).
- **Parameter:**
    - `fd`: Der Dateideskriptor (z. B. `1` für `stdout`).
    - `buf`: Ein Puffer mit den zu schreibenden Daten.
    - `count`: Die Anzahl der zu schreibenden Bytes.

---
Beispiel 
```sh
write(1, "drwxr-xr-x 3 yzh yzh     4096 Au"..., 47drwxr-xr-x 3 yzh yzh     4096 Aug 30 18:21 aws
) = 47
```

Der `write`-Aufruf in diesem Beispiel schreibt die Zeile `drwxr-xr-x 3 yzh yzh 4096 Aug 30 18:21 aws` auf stdout. Dies ist ein zentraler Teil der Funktionsweise von `ls -l`, da es sicherstellt, dass die gesammelten und formatierten Dateiinformationen dem Benutzer angezeigt werden.


Parameter 
- **Dateideskriptor (FD):**
    - **`1`**: Der Dateideskriptor `1` steht für den Standardausgabekanal (stdout). Dies bedeutet, dass die Ausgabe auf dem Terminal erscheint.
- **Inhalt des Puffers:**
    - **`"drwxr-xr-x 3 yzh yzh 4096 Aug 30 18:21 aws\n"`**:
        - Dieser String repräsentiert die formatierte Ausgabe einer Verzeichniszeile von `ls -l`. Die Zeile enthält folgende Informationen:
- **Größe des Inhalts:**
    - **`47`**: Die Anzahl der Bytes, die geschrieben werden sollen. Dies entspricht der Länge des Strings einschließlich des Zeilenumbruchs (`\n`).

**Rückgabewert:**
- **`= 47`**: Gibt an, dass 47 Bytes erfolgreich geschrieben wurden.


### 1.3.2 `close`

- **Funktion:** Schließt einen geöffneten Dateideskriptor.
- **Parameter:**
    - `fd`: Der zu schließende Dateideskriptor.

----

Beispiel 
`close(1) = 0`

Der Systemaufruf `close(1)` schließt die **Dateideskriptor-Nummer 1**, die in der Regel dem **Standardausgabestream (stdout)** entspricht.
- **Bedeutung**: Das bedeutet, dass der Standardausgabestream geschlossen wird, und keine weitere Ausgabe mehr auf die Konsole (oder das Terminal) geschrieben werden kann. Nach dem Schließen von `stdout` wird jede weitere Ausgabe, die an diesen Stream gerichtet ist, verworfen oder könnte in eine Datei umgeleitet werden, falls dies zuvor konfiguriert wurde.
- **Systemaufruf im Detail**:
    - `close(1)` bewirkt, dass der Dateideskriptor 1 (Standardausgabe) geschlossen wird.
    - Der Rückgabewert `= 0` bedeutet, dass der Aufruf erfolgreich war.




## 1.4 Ausgabe von `strace ls -l `

```sh
yzh@BNB23350:~$ ls -l
total 55076
drwxr-xr-x 3 yzh yzh     4096 Aug 30 18:21 aws
-rwxr-xr-x 1 yzh yzh 56381592 Sep  2 21:29 kubectl
-rw-r--r-- 1 yzh yzh       64 Sep  2 21:29 kubectl.sha256
-rw-r--r-- 1 yzh yzh       33 Oct  7 11:52 test



yzh@BNB23350:~$ strace ls -l
execve("/usr/bin/ls", ["ls", "-l"], 0x7ffc07daf008 /* 28 vars */) = 0
brk(NULL)                               = 0x563c82e1c000
arch_prctl(0x3001 /* ARCH_??? */, 0x7ffea4a5e580) = -1 EINVAL (Invalid argument)
mmap(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f324850e000
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=18699, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 18699, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f3248509000
close(3)                                = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libselinux.so.1", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\0\0\0\0\0\0\0\0"..., 832) = 832
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=166280, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 177672, PROT_READ, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7f32484dd000
mprotect(0x7f32484e3000, 139264, PROT_NONE) = 0
mmap(0x7f32484e3000, 106496, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x6000) = 0x7f32484e3000
mmap(0x7f32484fd000, 28672, PROT_READ, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x20000) = 0x7f32484fd000
mmap(0x7f3248505000, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x27000) = 0x7f3248505000
mmap(0x7f3248507000, 5640, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7f3248507000
close(3)                                = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\3\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0P\237\2\0\0\0\0\0"..., 832) = 832
pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 784, 64) = 784
pread64(3, "\4\0\0\0 \0\0\0\5\0\0\0GNU\0\2\0\0\300\4\0\0\0\3\0\0\0\0\0\0\0"..., 48, 848) = 48
pread64(3, "\4\0\0\0\24\0\0\0\3\0\0\0GNU\0I\17\357\204\3$\f\221\2039x\324\224\323\236S"..., 68, 896) = 68
newfstatat(3, "", {st_mode=S_IFREG|0755, st_size=2220400, ...}, AT_EMPTY_PATH) = 0
pread64(3, "\6\0\0\0\4\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0@\0\0\0\0\0\0\0"..., 784, 64) = 784
mmap(NULL, 2264656, PROT_READ, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7f32482b4000
mprotect(0x7f32482dc000, 2023424, PROT_NONE) = 0
mmap(0x7f32482dc000, 1658880, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x28000) = 0x7f32482dc000
mmap(0x7f3248471000, 360448, PROT_READ, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x1bd000) = 0x7f3248471000
mmap(0x7f32484ca000, 24576, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x215000) = 0x7f32484ca000
mmap(0x7f32484d0000, 52816, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x7f32484d0000
close(3)                                = 0
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libpcre2-8.so.0", O_RDONLY|O_CLOEXEC) = 3
read(3, "\177ELF\2\1\1\0\0\0\0\0\0\0\0\0\3\0>\0\1\0\0\0\0\0\0\0\0\0\0\0"..., 832) = 832
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=613064, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 615184, PROT_READ, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0x7f324821d000
mmap(0x7f324821f000, 438272, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x2000) = 0x7f324821f000
mmap(0x7f324828a000, 163840, PROT_READ, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x6d000) = 0x7f324828a000
mmap(0x7f32482b2000, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3, 0x94000) = 0x7f32482b2000
close(3)                                = 0
mmap(NULL, 12288, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f324821a000
arch_prctl(ARCH_SET_FS, 0x7f324821a800) = 0
set_tid_address(0x7f324821aad0)         = 64
set_robust_list(0x7f324821aae0, 24)     = 0
rseq(0x7f324821b1a0, 0x20, 0, 0x53053053) = 0
mprotect(0x7f32484ca000, 16384, PROT_READ) = 0
mprotect(0x7f32482b2000, 4096, PROT_READ) = 0
mprotect(0x7f3248505000, 4096, PROT_READ) = 0
mprotect(0x563c5c5c4000, 4096, PROT_READ) = 0
mprotect(0x7f3248548000, 8192, PROT_READ) = 0
prlimit64(0, RLIMIT_STACK, NULL, {rlim_cur=8192*1024, rlim_max=RLIM64_INFINITY}) = 0
munmap(0x7f3248509000, 18699)           = 0
statfs("/sys/fs/selinux", 0x7ffea4a5e5c0) = -1 ENOENT (No such file or directory)
statfs("/selinux", 0x7ffea4a5e5c0)      = -1 ENOENT (No such file or directory)
getrandom("\x6d\x97\xb7\x9f\xf8\x6a\x64\xef", 8, GRND_NONBLOCK) = 8
brk(NULL)                               = 0x563c82e1c000
brk(0x563c82e3d000)                     = 0x563c82e3d000
openat(AT_FDCWD, "/proc/filesystems", O_RDONLY|O_CLOEXEC) = 3
newfstatat(3, "", {st_mode=S_IFREG|0444, st_size=0, ...}, AT_EMPTY_PATH) = 0
read(3, "nodev\tsysfs\nnodev\ttmpfs\nnodev\tbd"..., 1024) = 478
read(3, "", 1024)                       = 0
close(3)                                = 0
access("/etc/selinux/config", F_OK)     = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/share/locale/locale.alias", O_RDONLY|O_CLOEXEC) = 3
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=2996, ...}, AT_EMPTY_PATH) = 0
read(3, "# Locale name alias data base.\n#"..., 4096) = 2996
read(3, "", 4096)                       = 0
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_IDENTIFICATION", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/lib/locale/C.utf8/LC_IDENTIFICATION", O_RDONLY|O_CLOEXEC) = 3
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=258, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 258, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f3248547000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache", O_RDONLY) = 3
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=27002, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 27002, PROT_READ, MAP_SHARED, 3, 0) = 0x7f3248213000
close(3)                                = 0
futex(0x7f32484cfa6c, FUTEX_WAKE_PRIVATE, 2147483647) = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_MEASUREMENT", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/lib/locale/C.utf8/LC_MEASUREMENT", O_RDONLY|O_CLOEXEC) = 3
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=23, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 23, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f324850d000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_TELEPHONE", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/lib/locale/C.utf8/LC_TELEPHONE", O_RDONLY|O_CLOEXEC) = 3
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=47, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 47, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f324850c000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_ADDRESS", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/lib/locale/C.utf8/LC_ADDRESS", O_RDONLY|O_CLOEXEC) = 3
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=127, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 127, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f324850b000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_NAME", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/lib/locale/C.utf8/LC_NAME", O_RDONLY|O_CLOEXEC) = 3
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=62, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 62, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f324850a000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_PAPER", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/lib/locale/C.utf8/LC_PAPER", O_RDONLY|O_CLOEXEC) = 3
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=34, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 34, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f3248509000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_MESSAGES", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/lib/locale/C.utf8/LC_MESSAGES", O_RDONLY|O_CLOEXEC) = 3
newfstatat(3, "", {st_mode=S_IFDIR|0755, st_size=4096, ...}, AT_EMPTY_PATH) = 0
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.utf8/LC_MESSAGES/SYS_LC_MESSAGES", O_RDONLY|O_CLOEXEC) = 3
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=48, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 48, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f3248212000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_MONETARY", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/lib/locale/C.utf8/LC_MONETARY", O_RDONLY|O_CLOEXEC) = 3
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=270, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 270, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f3248211000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_COLLATE", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/lib/locale/C.utf8/LC_COLLATE", O_RDONLY|O_CLOEXEC) = 3
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=1406, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 1406, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f3248210000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_TIME", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/lib/locale/C.utf8/LC_TIME", O_RDONLY|O_CLOEXEC) = 3
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=3360, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 3360, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f324820f000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_NUMERIC", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/lib/locale/C.utf8/LC_NUMERIC", O_RDONLY|O_CLOEXEC) = 3
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=50, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 50, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f324820e000
close(3)                                = 0
openat(AT_FDCWD, "/usr/lib/locale/C.UTF-8/LC_CTYPE", O_RDONLY|O_CLOEXEC) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/lib/locale/C.utf8/LC_CTYPE", O_RDONLY|O_CLOEXEC) = 3
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=353616, ...}, AT_EMPTY_PATH) = 0
mmap(NULL, 353616, PROT_READ, MAP_PRIVATE, 3, 0) = 0x7f32481b7000
close(3)                                = 0
ioctl(1, TCGETS, {B38400 opost isig icanon echo ...}) = 0
ioctl(1, TIOCGWINSZ, {ws_row=28, ws_col=108, ws_xpixel=0, ws_ypixel=0}) = 0
openat(AT_FDCWD, "/usr/share/locale/C.UTF-8/LC_TIME/coreutils.mo", O_RDONLY) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/share/locale/C.utf8/LC_TIME/coreutils.mo", O_RDONLY) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/share/locale/C/LC_TIME/coreutils.mo", O_RDONLY) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, ".", O_RDONLY|O_NONBLOCK|O_CLOEXEC|O_DIRECTORY) = 3
newfstatat(3, "", {st_mode=S_IFDIR|0750, st_size=4096, ...}, AT_EMPTY_PATH) = 0
getdents64(3, 0x563c82e25930 /* 28 entries */, 32768) = 920
statx(AT_FDCWD, "kubectl.sha256", AT_STATX_SYNC_AS_STAT|AT_SYMLINK_NOFOLLOW, STATX_MODE|STATX_NLINK|STATX_UID|STATX_GID|STATX_MTIME|STATX_SIZE, {stx_mask=STATX_BASIC_STATS|STATX_MNT_ID, stx_attributes=0, stx_mode=S_IFREG|0644, stx_size=64, ...}) = 0
lgetxattr("kubectl.sha256", "security.selinux", 0x563c82e2d940, 255) = -1 ENODATA (No data available)
getxattr("kubectl.sha256", "system.posix_acl_access", NULL, 0) = -1 ENODATA (No data available)
socket(AF_UNIX, SOCK_STREAM|SOCK_CLOEXEC|SOCK_NONBLOCK, 0) = 4
connect(4, {sa_family=AF_UNIX, sun_path="/var/run/nscd/socket"}, 110) = -1 ENOENT (No such file or directory)
close(4)                                = 0
socket(AF_UNIX, SOCK_STREAM|SOCK_CLOEXEC|SOCK_NONBLOCK, 0) = 4
connect(4, {sa_family=AF_UNIX, sun_path="/var/run/nscd/socket"}, 110) = -1 ENOENT (No such file or directory)
close(4)                                = 0
newfstatat(AT_FDCWD, "/etc/nsswitch.conf", {st_mode=S_IFREG|0644, st_size=510, ...}, 0) = 0
newfstatat(AT_FDCWD, "/", {st_mode=S_IFDIR|0755, st_size=4096, ...}, 0) = 0
openat(AT_FDCWD, "/etc/nsswitch.conf", O_RDONLY|O_CLOEXEC) = 4
newfstatat(4, "", {st_mode=S_IFREG|0644, st_size=510, ...}, AT_EMPTY_PATH) = 0
read(4, "# /etc/nsswitch.conf\n#\n# Example"..., 4096) = 510
read(4, "", 4096)                       = 0
newfstatat(4, "", {st_mode=S_IFREG|0644, st_size=510, ...}, AT_EMPTY_PATH) = 0
close(4)                                = 0
openat(AT_FDCWD, "/etc/passwd", O_RDONLY|O_CLOEXEC) = 4
newfstatat(4, "", {st_mode=S_IFREG|0644, st_size=1414, ...}, AT_EMPTY_PATH) = 0
lseek(4, 0, SEEK_SET)                   = 0
read(4, "root:x:0:0:root:/root:/bin/bash\n"..., 4096) = 1414
close(4)                                = 0
socket(AF_UNIX, SOCK_STREAM|SOCK_CLOEXEC|SOCK_NONBLOCK, 0) = 4
connect(4, {sa_family=AF_UNIX, sun_path="/var/run/nscd/socket"}, 110) = -1 ENOENT (No such file or directory)
close(4)                                = 0
socket(AF_UNIX, SOCK_STREAM|SOCK_CLOEXEC|SOCK_NONBLOCK, 0) = 4
connect(4, {sa_family=AF_UNIX, sun_path="/var/run/nscd/socket"}, 110) = -1 ENOENT (No such file or directory)
close(4)                                = 0
newfstatat(AT_FDCWD, "/etc/nsswitch.conf", {st_mode=S_IFREG|0644, st_size=510, ...}, 0) = 0
openat(AT_FDCWD, "/etc/group", O_RDONLY|O_CLOEXEC) = 4
newfstatat(4, "", {st_mode=S_IFREG|0644, st_size=769, ...}, AT_EMPTY_PATH) = 0
lseek(4, 0, SEEK_SET)                   = 0
read(4, "root:x:0:\ndaemon:x:1:\nbin:x:2:\ns"..., 4096) = 769
close(4)                                = 0
statx(AT_FDCWD, "kubectl", AT_STATX_SYNC_AS_STAT|AT_SYMLINK_NOFOLLOW, STATX_MODE|STATX_NLINK|STATX_UID|STATX_GID|STATX_MTIME|STATX_SIZE, {stx_mask=STATX_BASIC_STATS|STATX_MNT_ID, stx_attributes=0, stx_mode=S_IFREG|0755, stx_size=56381592, ...}) = 0
lgetxattr("kubectl", "security.selinux", 0x563c82e2e020, 255) = -1 ENODATA (No data available)
getxattr("kubectl", "system.posix_acl_access", NULL, 0) = -1 ENODATA (No data available)
statx(AT_FDCWD, "test", AT_STATX_SYNC_AS_STAT|AT_SYMLINK_NOFOLLOW, STATX_MODE|STATX_NLINK|STATX_UID|STATX_GID|STATX_MTIME|STATX_SIZE, {stx_mask=STATX_BASIC_STATS|STATX_MNT_ID, stx_attributes=0, stx_mode=S_IFREG|0644, stx_size=33, ...}) = 0
lgetxattr("test", "security.selinux", 0x563c82e2e150, 255) = -1 ENODATA (No data available)
getxattr("test", "system.posix_acl_access", NULL, 0) = -1 ENODATA (No data available)
statx(AT_FDCWD, "aws", AT_STATX_SYNC_AS_STAT|AT_SYMLINK_NOFOLLOW, STATX_MODE|STATX_NLINK|STATX_UID|STATX_GID|STATX_MTIME|STATX_SIZE, {stx_mask=STATX_BASIC_STATS|STATX_MNT_ID, stx_attributes=0, stx_mode=S_IFDIR|0755, stx_size=4096, ...}) = 0
lgetxattr("aws", "security.selinux", 0x563c82e2e280, 255) = -1 ENODATA (No data available)
getxattr("aws", "system.posix_acl_access", NULL, 0) = -1 ENODATA (No data available)
getxattr("aws", "system.posix_acl_default", NULL, 0) = -1 ENODATA (No data available)
getdents64(3, 0x563c82e25930 /* 0 entries */, 32768) = 0
close(3)                                = 0
openat(AT_FDCWD, "/usr/share/locale/C.UTF-8/LC_MESSAGES/coreutils.mo", O_RDONLY) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/share/locale/C.utf8/LC_MESSAGES/coreutils.mo", O_RDONLY) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/share/locale/C/LC_MESSAGES/coreutils.mo", O_RDONLY) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/share/locale-langpack/C.UTF-8/LC_MESSAGES/coreutils.mo", O_RDONLY) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/share/locale-langpack/C.utf8/LC_MESSAGES/coreutils.mo", O_RDONLY) = -1 ENOENT (No such file or directory)
openat(AT_FDCWD, "/usr/share/locale-langpack/C/LC_MESSAGES/coreutils.mo", O_RDONLY) = -1 ENOENT (No such file or directory)
newfstatat(1, "", {st_mode=S_IFCHR|0620, st_rdev=makedev(0x88, 0), ...}, AT_EMPTY_PATH) = 0
write(1, "total 55076\n", 12total 55076
)           = 12
openat(AT_FDCWD, "/etc/localtime", O_RDONLY|O_CLOEXEC) = 3
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=2298, ...}, AT_EMPTY_PATH) = 0
newfstatat(3, "", {st_mode=S_IFREG|0644, st_size=2298, ...}, AT_EMPTY_PATH) = 0
read(3, "TZif2\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\t\0\0\0\t\0\0\0\0"..., 4096) = 2298
lseek(3, -1449, SEEK_CUR)               = 849
read(3, "TZif2\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\0\t\0\0\0\t\0\0\0\0"..., 4096) = 1449
close(3)                                = 0
write(1, "drwxr-xr-x 3 yzh yzh     4096 Au"..., 47drwxr-xr-x 3 yzh yzh     4096 Aug 30 18:21 aws
) = 47
write(1, "-rwxr-xr-x 1 yzh yzh 56381592 Se"..., 51-rwxr-xr-x 1 yzh yzh 56381592 Sep  2 21:29 kubectl
) = 51
write(1, "-rw-r--r-- 1 yzh yzh       64 Se"..., 58-rw-r--r-- 1 yzh yzh       64 Sep  2 21:29 kubectl.sha256
) = 58
write(1, "-rw-r--r-- 1 yzh yzh       33 Oc"..., 48-rw-r--r-- 1 yzh yzh       33 Oct  7 11:52 test
) = 48
close(1)                                = 0
close(2)                                = 0
exit_group(0)                           = ?
+++ exited with 0 +++
```



# 2 B: lsmod and Kernel Module

Führen Sie auf Ihrem Linux-System lsmod aus. 
Wählen Sie zufällig 3 Module. 
Ermitteln Sie, welchem Zweck das Modul dient, und geben Sie einen Link zum Quelltext dieses Moduls an. 
Achten Sie dabei auf die korrekte Version des Quelltexts.


## 2.1 Wie man die Version des Quellcodes ermittelst

```
yzh@BNB23350:~$ uname -r  
6.8.0-35-generic
```

Dieser Befehl gibt dir die Version des aktuell laufenden Kernels zurück: 6.8.0-35-generic

## 2.2 **`tls` (Transport Layer Security)**
- **Zweck**: Das Modul `tls` implementiert den Transport Layer Security (TLS)-Protokollstack. TLS wird verwendet, um eine verschlüsselte Kommunikation zwischen Computern zu gewährleisten (z.B. in HTTPS-Verbindungen).
- **Link zum Quelltext**:
    - Das TLS-Modul ist Teil des Linux-Kernels und wird über den Git-Repository des Linux-Kernels verwaltet.
    - Quelle: https://github.com/torvalds/linux/tree/v6.8/net/tls


## 2.3 **`bnep` (Bluetooth Network Encapsulation Protocol)**

- **Zweck**: Das `bnep`-Modul ermöglicht das **Bluetooth Network Encapsulation Protocol** (BNEP). Dieses Modul wird verwendet, um Netzwerkdaten über eine Bluetooth-Verbindung zu übertragen. BNEP ermöglicht es, Netzwerkkommunikation über Bluetooth (z. B. zwischen einem Computer und einem Smartphone) durchzuführen. Eine typische Anwendung von BNEP ist die Verwendung von **Bluetooth PAN (Personal Area Network)**, das es Geräten ermöglicht, sich zu einem Netzwerk über Bluetooth zu verbinden.
    
- **Link zum Quelltext**:
    - Das `bnep`-Modul ist Teil des **Linux-Bluetooth-Subsystems**. Der Quellcode für die Bluetooth-Stacks und das `bnep`-Modul ist im Linux-Kernel-Repository enthalten.
    - Quelle: https://github.com/torvalds/linux/tree/v6.8/net/bluetooth/bnep

## 2.4 **`psmouse` (PS/2 Mouse Driver)**

- **Zweck**: Das `psmouse`-Modul ist ein Treiber für **PS/2-Mäuse** und Touchpads. Es ermöglicht den Betrieb von PS/2-Mäusen, die über den PS/2-Anschluss mit dem Computer verbunden sind. Der Treiber bietet Unterstützung für grundlegende Mausfunktionen und erweiterte Features wie Scrollen und das Erkennen von Touchpads. Auch modernere Geräte, wie beispielsweise USB-Mäuse, können von diesem Modul profitieren, wenn sie eine PS/2-Emulation verwenden.
    
- **Link zum Quelltext**:
    - Der Quellcode des `psmouse`-Moduls ist Teil des **Linux-Kernels**.
    - Du findest den Quellcode hier: https://github.com/torvalds/linux/tree/v6.8/drivers/input/mouse


## 2.5 Ausgabe von `lsmod`

```sh
yzh@BNB23350:~$ lsmod  
Module                  Size  Used by  
tls                   151552  0  
snd_seq_dummy          12288  0  
snd_hrtimer            12288  1  
snd_seq_midi           24576  0  
snd_seq_midi_event     16384  1 snd_seq_midi  
snd_rawmidi            57344  1 snd_seq_midi  
snd_seq               118784  9 snd_seq_midi,snd_seq_midi_event,snd_seq_dummy  
snd_seq_device         16384  3 snd_seq,snd_seq_midi,snd_rawmidi  
snd_timer              49152  2 snd_seq,snd_hrtimer  
snd                   147456  6 snd_seq,snd_seq_device,snd_timer,snd_rawmidi  
soundcore              16384  1 snd  
bnep                   32768  2  
intel_rapl_msr         20480  0  
intel_rapl_common      40960  1 intel_rapl_msr  
intel_uncore_frequency_common    16384  0  
intel_pmc_core        118784  0  
intel_vsec             20480  1 intel_pmc_core  
pmt_telemetry          16384  1 intel_pmc_core  
pmt_class              16384  1 pmt_telemetry  
crct10dif_pclmul       12288  1  
polyval_clmulni        12288  0  
polyval_generic        12288  1 polyval_clmulni  
ghash_clmulni_intel    16384  0  
sha256_ssse3           32768  0  
sha1_ssse3             32768  0  
aesni_intel           356352  0  
crypto_simd            16384  1 aesni_intel  
cryptd                 28672  2 crypto_simd,ghash_clmulni_intel  
vmw_balloon            28672  0  
rapl                   20480  0  
btusb                  77824  0  
btrtl                  36864  1 btusb  
btintel                57344  1 btusb  
btbcm                  24576  1 btusb  
btmtk                  16384  1 btusb  
bluetooth            1032192  13 btrtl,btmtk,btintel,btbcm,bnep,btusb  
ecdh_generic           16384  1 bluetooth  
ecc                    45056  1 ecdh_generic  
i2c_piix4              32768  0  
qrtr                   53248  4  
input_leds             12288  0  
joydev                 32768  0  
mac_hid                12288  0  
serio_raw              20480  0  
vsock_loopback         12288  0  
vmw_vsock_virtio_transport_common    61440  1 vsock_loopback  
vmw_vsock_vmci_transport    49152  2  
vsock                  65536  7 vmw_vsock_virtio_transport_common,vsock_loopback,vmw_vsock_vmci_transport  
vmw_vmci              106496  2 vmw_balloon,vmw_vsock_vmci_transport  
binfmt_misc            24576  1  
vmwgfx                442368  4  
drm_ttm_helper         12288  1 vmwgfx  
ttm                   114688  2 vmwgfx,drm_ttm_helper  
msr                    12288  0  
parport_pc             53248  0  
ppdev                  24576  0  
lp                     28672  0  
parport                77824  3 parport_pc,lp,ppdev  
efi_pstore             12288  0  
nfnetlink              20480  1  
dmi_sysfs              24576  0  
ip_tables              36864  0  
x_tables               69632  1 ip_tables  
autofs4                57344  2  
hid_generic            12288  0  
crc32_pclmul           12288  0  
psmouse               217088  0  
usbhid                 77824  0  
hid                   184320  2 usbhid,hid_generic  
e1000                 180224  0  
ahci                   49152  0  
mptspi                 24576  1  
mptscsih               53248  1 mptspi  
mptbase               122880  2 mptspi,mptscsih  
libahci                57344  1 ahci  
scsi_transport_spi     40960  1 mptspi  
pata_acpi              12288  0
```

