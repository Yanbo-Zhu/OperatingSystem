打旗语，发信号
信号量

Ein **Semaphor** ist eine Verallgemeinerung des Mutex-Konzepts. Er besteht aus einem ganzzahligen Zähler, dessen Wert nie negativ werden darf und einer Warteschlange für Prozesse/Threads. Es gibt zwei atomare Zugriffsoperationen:

- P(): Wenn der Zähler = 0 ist, blockiert der aufrufende Prozess und wartet auf einen positiven Zählerwert. Dann wird der Zähler dekrementiert 递减. 见到0 再次被堵塞 
- V(): Inkrementiert 递增 den Zähler und weckt ggf. einen auf den Semaphor wartenden Prozess auf.

Die Arbeitsweise von Semaphoren kennen Sie aus dem Modul _Computerarchitektur und Betriebssysteme_. Lesen Sie die Details bei Bedarf dort oder auf [Wikipedia: Semaphore (Programming)](https://en.wikipedia.org/wiki/Semaphore_(programming)) nach.

Mit Hilfe von Semaphoren kann man viele Synchronisationsprobleme, die über den einfachen Schutz eines kritischen Abschnitts hinausgehen, elegant lösen. Beispiele dafür gibt es in den [Übungen zu dieser Einheit](https://moodle.oncampus.de/modules/ir866/onmod/threads/exercises.html).

---

Ein **binärer Semaphor** (dessen Zähler nur die Werte 0 oder 1 annehmen kann und mit 1 initialisiert wird) entspricht funktional einem Mutex. Bei Mutexen gibt es aber zusätzlich die Einschränkung, dass sie nur von dem Prozess/Thread freigegeben werden können, der sie vorher belegt hat. Für Semaphore gilt dies nicht.


# 1 自己查解释 

是一种用于多线程或多进程同步的机制，可以用来控制对共享资源的访问。它通过维护一个计数器，来跟踪可用资源的数量，从而实现对资源的协调管理。

信号量是实现多线程编程中同步和资源共享的重要工具。它通过计数器和简单的P/V操作，可以灵活地管理共享资源的访问，同时确保线程间的安全与协作。
## 1.1 **信号量的特点**

1. **计数器机制**  
    信号量的核心是一个整型计数器：
    - 计数器的初始值表示资源的最大可用数量。
    - 每当一个线程或进程请求资源时，计数器减1。
    - 每当一个线程或进程释放资源时，计数器加1。
    - 当计数器为0时，请求资源的线程会进入等待状态，直到有资源可用。
2. **两种类型的信号量**
    - **二元信号量（Binary Semaphore）**  
        类似于互斥锁（Mutex），计数器只有0和1两个状态，通常用于实现互斥访问。
    - **计数信号量（Counting Semaphore）**  
        计数器可以是大于1的任意正整数，表示可以允许多个线程同时访问一定数量的资源。
3. **同步与互斥**
    - **同步**：信号量可以协调多个线程在某种顺序下执行，确保线程间的协作。
    - **互斥**：当信号量的计数值限制为1时，它可以实现类似于互斥锁的功能。

## 1.2 **信号量的主要操作**

1. **P操作（Proberen，尝试获取）**
    - 如果计数器大于0，计数器减1，线程成功获取资源。
    - 如果计数器为0，线程进入等待状态，直到计数器大于0。
2. **V操作（Verhogen，释放资源）**
    - 计数器加1，如果有等待线程，会唤醒一个等待线程。

## 1.3 **信号量的优缺点**

- **优点**：
    - 可同时管理多个共享资源。
    - 实现线程间的同步与互斥，功能更强大。
- **缺点**：
    - 实现和使用相对复杂，可能导致死锁或资源浪费。


## 1.4 **信号量的使用场景**

1. **资源计数**：  
    用于限制访问共享资源的线程数量，例如控制对数据库连接池的并发访问。
2. **线程同步**：  
    确保多个线程按照一定顺序执行，例如生产者-消费者模型。

# 2 例子，用于实现生产者-消费者模型：

```c
#include <stdio.h>
#include <pthread.h>
#include <semaphore.h>

#define BUFFER_SIZE 5

int buffer[BUFFER_SIZE];
int count = 0;

sem_t empty_slots;  // 表示空闲槽位
sem_t full_slots;   // 表示已占用槽位
pthread_mutex_t mutex;  // 保护共享资源的互斥锁

void *producer(void *arg) {
    for (int i = 0; i < 10; i++) {
        sem_wait(&empty_slots);  // 等待空闲槽位
        pthread_mutex_lock(&mutex);  // 加锁
        buffer[count++] = i;  // 放入数据
        printf("Produced: %d\n", i);
        pthread_mutex_unlock(&mutex);  // 解锁
        sem_post(&full_slots);  // 增加已占用槽位
    }
    return NULL;
}

void *consumer(void *arg) {
    for (int i = 0; i < 10; i++) {
        sem_wait(&full_slots);  // 等待已占用槽位
        pthread_mutex_lock(&mutex);  // 加锁
        int item = buffer[--count];  // 取出数据
        printf("Consumed: %d\n", item);
        pthread_mutex_unlock(&mutex);  // 解锁
        sem_post(&empty_slots);  // 增加空闲槽位
    }
    return NULL;
}

int main() {
    pthread_t prod, cons;
    sem_init(&empty_slots, 0, BUFFER_SIZE);
    sem_init(&full_slots, 0, 0);
    pthread_mutex_init(&mutex, NULL);

    pthread_create(&prod, NULL, producer, NULL);
    pthread_create(&cons, NULL, consumer, NULL);

    pthread_join(prod, NULL);
    pthread_join(cons, NULL);

    sem_destroy(&empty_slots);
    sem_destroy(&full_slots);
    pthread_mutex_destroy(&mutex);

    return 0;
}

```



# 3 POSIX Semaphore

Der POSIX-Standard definiert ein API für Semaphore. Es unterscheidet 2 Typen von Semaphoren:
- _Named Semaphores_ haben einen systemweit eindeutigen Namen, der mit `/` beginnt. Sie sind für _alle Prozesse im System_ unter diesem Namen sichtbar.
- _Unnamed Semaphores_ existieren in einem zwischen verschiedenen Prozessen/Threads geteilten Speicherbereich, z.B. einer globalen Variable des Programms. Sie sind nicht systemweit sichtbar.

Der Zugriff auf Semaphore kann wie bei Dateien nach Benutzern und Gruppen eingeschränkt werden. Dazu legt man für jedem neuen Semaphor die Zugriffsrechte der Kategorien _user/group/other_ fest.



Die wichtigsten Funktionen des POSIX-Semaphor-APIs sind:

|                  |                                                                         |
| ---------------- | ----------------------------------------------------------------------- |
| `sem_open()`     | named Semaphore initialisieren                                          |
| `sem_init()`     | unnamed Semaphore initialisieren                                        |
| `sem_wait()`     | entspricht P(): Zähler >0 dekrementieren, sonst blockieren und warten   |
| `sem_post()`     | entspricht V(): Zähler inkrementieren, ggf. wartenden Prozess aufwecken |
| `sem_getvalue()` | aktuellen Semaphor-Wert lesen                                           |


