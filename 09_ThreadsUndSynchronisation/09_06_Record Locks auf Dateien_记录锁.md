Record Locks
- exclusive record:  exclusive record lock
- shared record 

Wenn mehrere Prozesse (nicht Threads!) auf eine gemeinsame Datei zugreifen, kann der Zugriff über spezielle Sperren synchronisiert werden. 

POSIX definiert dafür zwei Mechanismen:
- Der System Call `fcntl()` (_file control_) ermöglicht es unter anderem, sogenante **record locks** in einer offenen Datei zu setzen.
    - Diese Locks beziehen sich jeweils auf einen ==durch Start- und Endposition bestimmten Abschnitt der Date==i. Sie können _exclusive_ (zum Schreiben) oder _shared_ (zum Lesen) sein. 
    - Ein _exclusive record lock_ wird nur vergeben, wenn kein _shared record_ auf demselben Abschnitt besteht. Das Locking mit `fcntl()` ist sehr flexibel, aber recht kompliziert zu bedienen. 
- Die POSIX-Bibliotheksfunktion `lockf()` stellt ein einfacheres API zur Verfügung, das in der Implementierung typischerweise intern auf `fcntl()` abgebildet wird.


Unter Linux gibt es auch das Kommando [flock](https://man7.org/linux/man-pages/man1/flock.1.html), mit dem man aus der Shell Dateien mit `flock()` sperren kann — allerdings nur komplett, nicht abschnittsweise.

Ein gesetztes _record lock_ **verhindert nicht** den Zugriff auf die Datei durch andere Prozesse, die selbst keines anfordern. POSIX spricht von _advisory record locking_.


# 1 中文解释 

是一种用于在多个进程（注意不是线程！）访问共享文件时，同步访问的机制。通过记录锁，可以==对文件的特定部分进行访问控制==，避免多个进程同时对文件的同一区域进行读写操作，从而确保数据的一致性和完整性。

记录锁是 POSIX 系统中提供的强大文件同步工具，通过灵活设置文件的锁定范围和类型，可以有效管理多个进程对文件的并发访问。`fcntl()` 提供了最大灵活性，而 `lockf()` 则更易于使用



 **记录锁的特点**
1. **两种类型的记录锁**
    - **独占记录锁（Exclusive Record Lock）**  
        用于写操作时的锁。只有当文件的某一区域没有任何共享锁（Shared Record Lock）存在时，才能成功申请独占锁。
    - **共享记录锁（Shared Record Lock）**  
        用于读操作时的锁。多个进程可以同时对同一区域设置共享锁，但不能与独占锁共存。
2. **锁的范围**
    - 锁的作用范围可以由文件的起始位置和结束位置指定，仅针对该范围内的数据有效。
    - 一个文件可以有多个独立的锁区间，分别被不同的进程锁定。
3. **进程级锁**
    - 记录锁是进程级的，线程间不能直接使用记录锁同步。

记录锁的工作原理
- **写操作时**：
    - 进程 A 设置独占锁后，进程 B 无法在同一区域设置共享锁或独占锁。
- **读操作时**：
    - 进程 A 设置共享锁后，进程 B 可以在同一区域设置共享锁，但无法设置独占锁。


记录锁的应用场景
- **文件并发访问控制**：当多个进程需要同时读取和写入同一文件时，通过记录锁可以避免数据冲突。
- **数据库文件锁定**：在数据库文件中，记录锁常用于确保事务的隔离性和一致性。
- **日志文件管理**：保证多个进程同时写入日志文件时不会相互干扰。


## 1.1 POSIX 提供的两种实现方式

**`fcntl()` 系统调用**
- `fcntl()` 提供了灵活的接口，可以通过它在文件的特定区域设置记录锁。
- 使用时可以指定锁的类型（独占或共享）、锁的起始位置和长度。
- 灵活性强，但操作复杂，通常需要编写额外的代码来处理锁定和解锁操作。
- 示例调用：

```c
struct flock lock;
lock.l_type = F_WRLCK;  // 写锁
lock.l_whence = SEEK_SET;
lock.l_start = 0;       // 锁定起始位置
lock.l_len = 100;       // 锁定长度
fcntl(fd, F_SETLK, &lock);

```


**`lockf()` 函数**
- `lockf()` 是一个更简单的接口，用于对文件的一部分或整个文件进行锁定。
- 通常是对 `fcntl()` 的封装，因此功能上不如 `fcntl()` 强大，但使用更加直观。

```
lockf(fd, F_LOCK, 100);  // 锁定从当前位置开始的100字节
```
