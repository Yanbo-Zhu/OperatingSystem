
# 1 一个新的shell被启动后会读取的信息 


Beim Start einer neuen Shell liest diese nacheinander verschiedene Dateien, die Scripts zur Initialisierung der Shell enthalten. Darin wird typischerweise
- das Prompt gesetzt
- der Suchpfad PATH definiert
- Shell-Optionen und relevante Environment-Variablen gesetzt
- Aliases definiert
- beim Login evtl. für die Benutzer-Session benötigte Hilfsdienste im Hintergrund gestartet

# 2 shell type

不同种shell会读取不同的信息 
1. _login-Shells_, die für einen Benutzer nach dem Einloggen zuerst gestartet werden
2. _interaktive Shells_, die vom Benutzer interaktiv in einem Terminal verwendet werden
3. _nicht-interaktive Shells_, die zur Ausführung von Shell Scripts gestartet werden

使用 `echo $0` 去判别shell种类
Mit `echo $0` können Sie herausfinden, welche Shell gerade läuft. Wenn der Name mit einem `-` beginnt (wie `-bash`), handelt es sich um eine login-Shell.



# 3 设置 bevorzugten Texteditor

Um Ihren bevorzugten Texteditor festzulegen, ergänzen Sie in `~/.profile` den Befehl
`export EDITOR=vim   # oder ein anderer Editor`

Die Environment-Variable EDITOR wird von vielen Tools gelesen, aus denen heraus ein Editor gestartet werden kann, z.B. `less`.


# 4 Bash Startup Files 

那些 Files 会被读取 在 bash启动的时候 
https://www.gnu.org/software/bash/manual/html_node/Bash-Startup-Files.html

In den benutzerspezifischen _startup files_ wie `~/.bashrc` und `~/.profile` können Sie individuelle Initialisierungen ergänzen.

This section describes how Bash executes its startup files. 


## 4.1 Invoked as an interactive login shell, or with --login

When Bash is invoked as an interactive login shell, or as a non-interactive shell with the --login option, it first reads and executes commands from the file `/etc/profile`, if that file exists. 
After reading that file, it looks for `~/.bash_profile, ~/.bash_login, and ~/.profile`, in that order, and reads and executes commands from the first one that exists and is readable. The --noprofile option may be used when the shell is started to inhibit this behavior.

When an interactive login shell exits, or a non-interactive login shell executes the `exit` builtin command, Bash reads and executes commands from the file `~/.bash_logout`, if it exists.

## 4.2 Invoked as an interactive non-login shell  (最长的情况)

When an interactive shell that is not a login shell is started, Bash reads and executes commands from `~/.bashrc`, if that file exists. This may be inhibited by using the `--norc` option. The `--rcfile` file option will force Bash to read and execute commands from file instead of ~/.bashrc.

So, typically, your `~/.bash_profile` contains the line

```
if [ -f ~/.bashrc ]; then . ~/.bashrc; fi
```

after (or before) any login-specific initializations.

## 4.3 Invoked non-interactively

如果 bash is started non-interactively ,  variable `BASH_ENV` 中 定义的 file 会被执行. 这些file中的命令会生效 

When Bash is started non-interactively, to run a shell script, for example, it looks for the variable `BASH_ENV` in the environment, expands its value if it appears there, and uses the expanded value as the name of a file to read and execute. Bash behaves as if the following command were executed:

```
if [ -n "$BASH_ENV" ]; then . "$BASH_ENV"; fi
```

but the value of the `PATH` variable is not used to search for the filename.

As noted above, if a non-interactive shell is invoked with the --login option, Bash attempts to read and execute commands from the login shell startup files.


## 4.4 Invoked with name `sh`

If Bash is invoked with the name `sh`, it tries to mimic the startup behavior of historical versions of `sh` as closely as possible, while conforming to the POSIX standard as well.

1  invoked as an interactive login she
When invoked as an interactive login shell, or as a non-interactive shell with the --login option, ==it first attempts to read and execute commands from /etc/profile and ~/.profile==, in that order. 
The --noprofile option may be used to inhibit this behavior. 

2 invoked as an interactive shell with the name `sh`
When invoked as an interactive shell with the name `sh`, Bash looks for the variable `ENV`, expands its value if it is defined, and uses the expanded value as the name of a file to read and execute. 

Since a shell invoked as `sh` does not attempt to read and execute commands from any other startup files, the --rcfile option has no effect. 

A non-interactive shell invoked with the name `sh` does not attempt to read any other startup files.

When invoked as `sh`, Bash enters POSIX mode after the startup files are read.




