# **Mastering Threads in C: A Ruthless Deep Dive**

## **1. Introduction to Threads**

### **1.1 What is a Thread?**

A **thread** is the **smallest unit of execution** within a process. Think of a process as a program in memory, and threads as independent lines of execution **inside that process**.

* A single process can have **one or more threads**.
* Threads **share the process's resources** (heap, global variables, open files) but have **their own stack and registers**.
* Threads are sometimes called **lightweight processes**.

#### Example Analogy:

Imagine a factory (process). Each worker (thread) works on a different task but shares the factory floor, tools, and raw materials.

---

### **1.2 Benefits of Threads**

1. **Concurrency**: Multiple tasks execute simultaneously.
2. **Resource Sharing**: Threads share memory space, so communication is easier.
3. **Responsiveness**: GUI or servers stay responsive.
4. **Efficiency**: Less overhead than creating multiple processes.

---

### **1.3 Types of Threads**

Threads are categorized based on **where they are managed**:

#### **1.3.1 User-Level Threads (ULT)**

* Managed entirely by **user-space libraries**.
* The OS doesn’t know about them.
* **Advantages**: Fast creation, context switch is cheap.
* **Disadvantages**: If one thread blocks, the entire process blocks.

#### **1.3.2 Kernel-Level Threads (KLT)**

* Managed by the **OS kernel**.
* **Advantages**: True concurrency on multiple cores; blocking one thread doesn’t block others.
* **Disadvantages**: Slower creation and context switching.

#### **1.3.3 Hybrid Threads**

* Combines **user-level management** with **kernel-level execution**.
* Examples: Solaris, Windows.

---

## **2. Thread Lifecycle**

A thread typically has **five states**:

1. **New** – Thread is created but not started.
2. **Runnable/Ready** – Thread is ready to run, waiting for CPU.
3. **Running** – Thread is executing instructions.
4. **Waiting/Blocked** – Thread waits for an event or resource.
5. **Terminated** – Thread finished execution.

---

## **3. Thread Programming in C**

In C, the most common API for threads is **POSIX Threads (pthreads)** on Linux/Unix.

### **3.1 pthread Basics**

Header:

```c
#include <pthread.h>
```

Core functions:

| Function           | Description                                      |
| ------------------ | ------------------------------------------------ |
| `pthread_create()` | Create a new thread                              |
| `pthread_exit()`   | Terminate calling thread                         |
| `pthread_join()`   | Wait for a thread to finish                      |
| `pthread_detach()` | Detach thread to reclaim resources automatically |
| `pthread_self()`   | Get current thread ID                            |

---

### **3.2 Creating a Thread**

```c
#include <stdio.h>
#include <pthread.h>
#include <stdlib.h>

void* thread_function(void* arg) {
    int num = *((int*)arg);
    printf("Thread is running. Value: %d\n", num);
    return NULL;
}

int main() {
    pthread_t thread;
    int value = 42;

    // Create a thread
    if(pthread_create(&thread, NULL, thread_function, &value)) {
        fprintf(stderr, "Error creating thread\n");
        return 1;
    }

    // Wait for thread to finish
    if(pthread_join(thread, NULL)) {
        fprintf(stderr, "Error joining thread\n");
        return 2;
    }

    printf("Thread finished execution\n");
    return 0;
}
```

**Explanation:**

1. `pthread_t thread;` – Thread handle (ID).
2. `pthread_create(&thread, NULL, thread_function, &value);`

   * `&thread` – store thread ID
   * `NULL` – default attributes
   * `thread_function` – function the thread executes
   * `&value` – argument passed to the thread
3. `pthread_join(thread, NULL);` – Wait for thread to finish.

---

### **3.3 Exiting Threads**

* `pthread_exit(void *retval)` – terminate current thread.
* The value can be collected by `pthread_join()`.

```c
void* thread_function(void* arg) {
    int* result = malloc(sizeof(int));
    *result = 123;
    pthread_exit(result);
}

int main() {
    pthread_t thread;
    void* retval;

    pthread_create(&thread, NULL, thread_function, NULL);
    pthread_join(thread, &retval);

    printf("Thread returned: %d\n", *((int*)retval));
    free(retval);
}
```

---

### **3.4 Detaching Threads**

* Detached threads **clean up resources automatically** after finishing.
* You **cannot join a detached thread**.

```c
pthread_detach(thread);
```

---

## **4. Thread Synchronization**

Threads share memory, so **race conditions** can occur.

### **4.1 Mutex (Mutual Exclusion)**

```c
pthread_mutex_t lock;

pthread_mutex_init(&lock, NULL); // Initialize

pthread_mutex_lock(&lock);   // Enter critical section
// critical code
pthread_mutex_unlock(&lock); // Exit critical section

pthread_mutex_destroy(&lock); // Cleanup
```

---

### **4.2 Condition Variables**

Threads can **wait for conditions** to be true.

```c
pthread_cond_t cond;
pthread_mutex_t lock;

pthread_cond_wait(&cond, &lock);   // Wait for condition
pthread_cond_signal(&cond);        // Wake one waiting thread
pthread_cond_broadcast(&cond);     // Wake all waiting threads
```

---

### **4.3 Read-Write Locks**

* Allow **multiple readers or a single writer**.
* Useful when reads are frequent, writes are rare.

```c
pthread_rwlock_t rwlock;
pthread_rwlock_rdlock(&rwlock); // reader
pthread_rwlock_wrlock(&rwlock); // writer
pthread_rwlock_unlock(&rwlock);
```

---

### **4.4 Semaphores**

* Control **access to limited resources**.

```c
#include <semaphore.h>

sem_t sem;
sem_init(&sem, 0, 3); // 3 resources

sem_wait(&sem);   // Acquire
sem_post(&sem);   // Release
sem_destroy(&sem);
```

---

## **5. Advanced Thread Topics**

### **5.1 Thread Scheduling**

* Linux threads can have **scheduling policies**:

  * `SCHED_OTHER` – default
  * `SCHED_FIFO` – first-in, first-out real-time
  * `SCHED_RR` – round-robin real-time

```c
struct sched_param param;
param.sched_priority = 10;
pthread_setschedparam(thread, SCHED_FIFO, &param);
```

### **5.2 Thread-Local Storage (TLS)**

* Each thread has **private variables**.

```c
__thread int thread_specific_data;
```

---

### **5.3 Deadlocks**

Occurs when **two or more threads wait for each other indefinitely**.

**Preventive strategies:**

1. Always acquire locks in a consistent order.
2. Use `pthread_mutex_trylock()`.
3. Use timeout locks.

---

### **5.4 Thread Pools**

* Reuse threads to **avoid overhead** of creation/destruction.
* Common in servers.

---

### **5.5 Thread Safety**

* Functions must be **reentrant** if accessed by multiple threads.
* Avoid global variables unless protected by mutexes.

---

## **6. Full Example: Producer-Consumer Problem**

```c
#include <stdio.h>
#include <pthread.h>
#include <semaphore.h>
#include <unistd.h>

#define BUFFER_SIZE 5
int buffer[BUFFER_SIZE];
int count = 0;

pthread_mutex_t lock;
sem_t empty, full;

void* producer(void* arg) {
    int item;
    for(int i=0; i<10; i++) {
        item = i;
        sem_wait(&empty);
        pthread_mutex_lock(&lock);

        buffer[count++] = item;
        printf("Produced: %d\n", item);

        pthread_mutex_unlock(&lock);
        sem_post(&full);
        sleep(1);
    }
    return NULL;
}

void* consumer(void* arg) {
    int item;
    for(int i=0; i<10; i++) {
        sem_wait(&full);
        pthread_mutex_lock(&lock);

        item = buffer[--count];
        printf("Consumed: %d\n", item);

        pthread_mutex_unlock(&lock);
        sem_post(&empty);
        sleep(2);
    }
    return NULL;
}

int main() {
    pthread_t prod, cons;
    pthread_mutex_init(&lock, NULL);
    sem_init(&empty, 0, BUFFER_SIZE);
    sem_init(&full, 0, 0);

    pthread_create(&prod, NULL, producer, NULL);
    pthread_create(&cons, NULL, consumer, NULL);

    pthread_join(prod, NULL);
    pthread_join(cons, NULL);

    pthread_mutex_destroy(&lock);
    sem_destroy(&empty);
    sem_destroy(&full);
}
```

This demonstrates **real multithreading**, **mutex**, **semaphore**, and **synchronization** together.

---

## **7. Key Takeaways for Mastery**

1. Threads are **independent paths of execution** within the same process.
2. Always manage **shared resources** carefully (mutexes, semaphores, etc.).
3. Use **thread-local storage** for per-thread variables.
4. Understand **thread scheduling and priorities**.
5. Avoid deadlocks and race conditions.
6. Real-world use: GUI apps, servers, simulations, multithreaded pipelines.


