
# 1 Lock

**Sperren (locks)** sind ein grundlegender Mechanismus zur Synchronisation. Eine Sperre kann 2 Zustände haben: _frei_ oder _belegt_. Sie ist also eine boolesche Variable, die von den beteiligten Prozessen/Threads geteilt wird. 

Ein Prozess/Thread kann 2 Aktionen auf der Sperre durchführen
1. Die Sperre belegen/anfordern (_acquire/lock_): Falls die Sperre frei ist, wird sie belegt. Falls nicht, wartet der Prozess/Thread zunächst, bis sie wieder frei ist.
2. Eine von ihm selbst belegte Sperre wieder freigeben (_release/unlock_).

Das Testen und Belegen der Sperre muss **atomar** (ununterbrechbar 不可被打断的) sein, damit die Sperre sicher funktioniert und keine neue Race Condition entsteht. Normalerweise wird dies mit Unterstützung der Hardware realisiert, z.B. mit speziellen CPU-Instruktionen.

Es gibt verschiedene Methoden, Sperren zu realisieren. Sie unterscheiden sich vor allem darin, wie auf eine aktuell belegte Sperre gewartet wird.


# 2 Spinlocks
自旋锁 
**自旋锁**是一种轻量级的锁机制，用于多线程或多进程编程中同步对共享资源的访问。它的特点是，当一个线程尝试获取锁而发现锁已被其他线程占用时，线程不会进入阻塞状态，而是会在一个循环中反复检查锁的状态，直到锁被释放为止。这种“忙等待”的行为称为“自旋”。

> Ein **Spinlock** ist eine Sperre mit _aktivem Warten_: Der anfordernde Prozess/Thread wartet in einer Schleife, bis die Sperre frei ist, und belegt sie dann.


Dies verschwendet natürlich CPU-Zeit auf unproduktive Arbeiten. Andererseits spart man sich den Verwaltungsaufwand, den aufrufenden Prozess/Thread zu blockieren und später (wenn die Sperre frei ist) wieder aufzuwecken.

Spinlocks werden für kurze kritische Abschnitte verwendet, bei denen die mittlere Wartezeit bis zum Freiwerden der Sperre sehr klein ist. In diesen Situationen sind sie sehr effizient.

Innerhalb des Kernels werden oft Spinlocks verwendet, z.B. zum Sperren der globalen Prozesstabelle, während der Scheduler einen Prozess einer bestimmten CPU zuteilt.


Wenn man in unserem Beispiel `counter.c` den Zähler ohne künstliche Verzögerung mit `counter++` hochzählt, besteht der kritische Abschnitt aus einer Sequenz von nur 3 CPU-Instruktionen:  
laden — inkrementieren — speichern.  
Das geht so schnell, dass ein aktives Warten _viel effizienter_ ist als einen Thread erst zu blockieren und später wieder aufzuwecken.

Hier würde man also ein Spinlock bevorzugen.


## 2.1 自己查的解释
**Spinlocks** (_自旋锁_) are a type of synchronization mechanism used in multithreading and multiprocessing to prevent multiple threads or processes from accessing a shared resource simultaneously in a way that causes data corruption or inconsistent behavior.
不同 thread 需要使用同一个资源, 如何实现轮流获取 

Key Characteristics of Spinlocks:
1. **Active Waiting**:
    - When a thread or process attempts to acquire a spinlock and finds it already held by another, it "spins" in a loop, continuously checking until the lock becomes available.
    - Unlike blocking locks, spinlocks do not put the thread to sleep during the wait.
2. **Efficiency for Short Waits**:
    - Spinlocks are most effective when the lock is held for a very short duration.
    - Context switching between threads is avoided, which can save overhead in certain scenarios.
3. **Busy-Waiting Drawback**:
    - While spinning, the thread consumes CPU cycles, which can lead to inefficiency if the lock is held for a longer time.
4. **Implementation**:
    - Spinlocks are typically implemented using atomic operations like `test-and-set`, `compare-and-swap`, or similar hardware-level primitives to ensure safe manipulation of the lock variable.

Pseudocode Example for a Spinlock:
```
// Shared spinlock variable
spinlock = 0  // 0 means unlocked, 1 means locked

function acquire_lock():
    while atomic_test_and_set(&spinlock) == 1:
        ; // Busy-wait (spin)

function release_lock():
    spinlock = 0  // Unlock

```

Advantages:
- Low overhead for short critical sections.
- Avoids the cost of putting threads to sleep and waking them up.

Disadvantages:
- Inefficient for long waits due to CPU time wastage.
- Not suitable for systems where thread scheduling fairness is critical.

Spinlocks are widely used in kernel-level programming (e.g., operating system kernels) and certain high-performance applications.

---

**自旋锁**是一种轻量级的锁机制，用于多线程或多进程编程中同步对共享资源的访问。它的特点是，当一个线程尝试获取锁而发现锁已被其他线程占用时，线程不会进入阻塞状态，而是会在一个循环中反复检查锁的状态，直到锁被释放为止。这种“忙等待”的行为称为“自旋”。

**自旋锁的特点：**
1. **忙等待（Busy Waiting）**  
    线程不会放弃CPU，而是持续检查锁的状态。这避免了线程切换的开销，但也可能浪费CPU资源。
    
2. **适用场景**  
    自旋锁适合用于**锁持有时间短**的场景，因为在锁占用时间较短的情况下，忙等待的开销小于线程切换的开销。
    
3. **不适合长时间锁持有**  
    如果锁被长时间占用，自旋锁会导致CPU资源浪费，降低系统性能。
    
4. **实现简单**  
    自旋锁通常依赖于底层的原子操作（如Test-and-Set或Compare-and-Swap）实现，能够高效地检查和更新锁的状态。
    

优缺点
- **优点**：
    
    - 避免了线程阻塞和切换的开销，适合短期锁定。
    - 实现简单，性能高效。
- **缺点**：
    
    - 在锁持有时间长时会浪费CPU资源。
    - 不适合高并发场景，容易导致“饥饿”问题。

## 2.2 Pthread-Spinlocks


Das pthread-API enthält einen Spinlock-Mechanismus. Die wichtigsten API-Funktionen dafür sind:

|   |   |
|---|---|
|`pthread_spin_init()`|initialisiert ein Spinlock|
|`pthread_spin_lock()`|belegt ein Spinlock (mit aktivem Warten)|
|`pthread_spin_unlock()`|gibt ein Spinlock frei|

```c
#include <stdio.h>
#include <pthread.h>
#include <stdatomic.h>

// 定义一个原子变量作为自旋锁
atomic_flag spinlock = ATOMIC_FLAG_INIT;

// 自旋锁的加锁函数
void spinlock_lock(atomic_flag *lock) {
    while (atomic_flag_test_and_set(lock)) {
        // 自旋，直到锁被释放
    }
}

// 自旋锁的解锁函数
void spinlock_unlock(atomic_flag *lock) {
    atomic_flag_clear(lock); // 释放锁
}

// 共享资源
int counter = 0;

// 线程任务
void *increment_counter(void *arg) {
    for (int i = 0; i < 1000000; i++) {
        spinlock_lock(&spinlock); // 加锁
        counter++;               // 修改共享资源
        spinlock_unlock(&spinlock); // 解锁
    }
    return NULL;
}

int main() {
    pthread_t thread1, thread2;

    // 创建两个线程
    pthread_create(&thread1, NULL, increment_counter, NULL);
    pthread_create(&thread2, NULL, increment_counter, NULL);

    // 等待线程完成
    pthread_join(thread1, NULL);
    pthread_join(thread2, NULL);

    // 输出最终计数器值
    printf("Final counter value: %d\n", counter);

    return 0;
}

```


# 3 Mutex  

互斥

> Ein **Mutex** (von _mutual exclusion_) ist eine _blockierende_ Sperre: Ein anfordernder Prozess oder Thread wird blockiert, wenn die Sperre belegt ist, und wieder aufgeweckt, sobald sie frei wird.

A **mutex** (short for _mutual exclusion_) is a _blocking_ lock: a requesting process or thread is blocked if the lock is occupied and will be awakened once the lock becomes available.

**互斥锁**（Mutex，来自“mutual exclusion”）是一种**阻塞**锁：当某个进程或线程尝试获取互斥锁时，如果锁已被其他线程占用，该线程会被阻塞，直到锁释放后 该线程才会被唤醒继续执行。

**互斥锁的作用**是确保多个线程或进程不会同时访问共享资源，从而避免竞争条件（Race Condition）和数据不一致的问题。

Zur Implementierung eines Mutex benötigt man zusätzlich zur Sperre eine _Warteschlange (queue)_ der darauf wartenden Prozesse/Threads.





## 3.1 Pthread-Mutexe

Das pthread-API stellt auch einen Mutex-Mechanismus bereit. Die API-Funktionen dafür sind analog zu denen für Spinlocks:

|   |   |
|---|---|
|`pthread_mutex_init()`|initialisiert ein Mutex|
|`pthread_mutex_lock()`|belegt ein Mutex (mit Blockieren)|
|`pthread_mutex_unlock()`|gibt ein Mutex frei|


Kopieren Sie das Programm `racecond.c` in die Datei `mutex.c`. Sichern Sie darin den kritischen Abschnitt mit einem Mutex ab. Testen Sie Ihre Lösung.


# 4 Mutex 和spinlock 的比较 

**互斥锁**（Mutex，来自“mutual exclusion”）是一种**阻塞**锁：当某个进程或线程尝试获取互斥锁时，如果锁已被其他线程占用，该线程会被阻塞，直到锁释放后 该线程才会被唤醒继续执行。

**自旋锁**是一种轻量级的锁机制，用于多线程或多进程编程中同步对共享资源的访问。它的特点是，当一个线程尝试获取锁而发现锁已被其他线程占用时，线程不会进入阻塞状态，而是会在一个循环中反复检查锁的状态，直到锁被释放为止。这种“忙等待”的行为称为“自旋”。

Für längere kritische Abschnitte sind Mutexe effizienter als Spinlocks, da der wartende Prozess/Thread keine CPU-Ressourcen verbraucht.

Spinlocks werden für kurze kritische Abschnitte verwendet, bei denen die mittlere Wartezeit bis zum Freiwerden der Sperre sehr klein ist. In diesen Situationen sind sie sehr effizient.

---

Die pthread-Mutexe und -Spinlocks sind primär zur Verwendung in Threads innerhalb eines Prozesses vorgesehen. Man kann beide Mechanismen prinzipiell auch Prozess-übergreifend verwenden, wenn man bei der Initialisierung das Attribut `PTHREAD_PROCESS_SHARED` setzt.

Dazu muss das Spinlock- oder Mutex-Objekt aber in einem [_shared memory_](https://moodle.oncampus.de/modules/ir866/onmod/ipc/shm.html)-Segment der beteiligten Prozesse gespeichert sein, damit diese überhaupt gemeinsam darauf zugreifen können.



