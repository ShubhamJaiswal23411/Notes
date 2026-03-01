
# 1️. What Is a Thread?

A **Thread** is:
> The smallest unit of execution within a process.
- A **process** = running program
- A **thread** = lightweight sub-process inside it

Example:
- Browser → multiple threads (UI, network, rendering)

---

# 2. What Is Multithreading?

Multithreading is:
> Running multiple threads concurrently within a single process.

---
## 1. Why Multithreading?
- Better CPU utilization
- Improved responsiveness (UI apps)
- Parallel task execution
- Background processing
- High-performance servers

---
## 2. Concurrency vs Parallelism

| Term        | Meaning                                     |
| ----------- | ------------------------------------------- |
| Concurrency | Multiple tasks making progress              |
| Parallelism | Tasks executing simultaneously (multi-core) |

Java supports both.

---

# 3️. Thread Creation in Java

There are 3 main ways:

---
##  1. Extending `Thread` Class

```java
class MyThread extends Thread {
    public void run() {
        System.out.println("Thread running");
    }
}

MyThread t = new MyThread();
t.start();
```

***Always call `start()`, not `run()`.***

---

##  2. Implementing `Runnable` (Recommended)

```java
class MyTask implements Runnable {
    public void run() {
        System.out.println("Task running");
    }
}

Thread t = new Thread(new MyTask());
t.start();
```
### Why Preferred?
- Java doesn’t support multiple inheritance
- Separation of task & thread
- Better design

---
##  3. Using Lambda (Java 8+)

```java
Thread t = new Thread(() -> {
    System.out.println("Running");
});
t.start();
```

---
# 4. Thread Lifecycle

A thread goes through these states:

```
NEW
RUNNABLE
BLOCKED
WAITING
TIMED_WAITING
TERMINATED
```

Important point : these are the actual state enum values and it doesn't contain RUNNING because java treats RUNNABLE as running or runnable state.

---
##  1. NEW
- Thread object created
- `start()` not called
##  2. RUNNABLE
- `start()` called
- Ready to run or running 
- Waiting for CPU
##  3. BLOCKED
- Waiting for monitor lock (synchronized block)
##  4. WAITING
- Waiting indefinitely
- Methods:
    - `wait()`
    - `join()`
    - `LockSupport.park()`
##  5. TIMED_WAITING
- Waiting for specific time
- Methods:
    - `sleep()`
    - `wait(timeout)`
    - `join(timeout)`
##  6. TERMINATED
- `run()` method completed
- ***Cannot restart thread***

---
# 5. Important Thread Methods

###  `start()`
- Creates new call stack
- Moves thread to RUNNABLE

###  `run()`
- Contains task logic
- Calling directly = normal method call (no new thread)

###  `sleep(milliseconds)`
- Pauses thread
- Does NOT release lock

###  `join()`
- Waits for another thread to finish\

###  `yield()`
- Suggests scheduler to give chance to other threads

###  `interrupt()`
- Signals thread to stop
- Sets interrupt flag

###  `isAlive()`
- Checks if thread is still running

# 6. Thread Scheduling
- Controlled by JVM + OS
- Uses **priority system**

```java
thread.setPriority(Thread.MAX_PRIORITY);
```

Priorities:
- 1 (MIN)
- 5 (NORM)
- 10 (MAX)
***Not guaranteed behavior (OS dependent)***

---
# 7. Synchronization & Thread Safety

When multiple threads access shared data → race conditions may occur.

Example:

```java
count++;
```

Not atomic.

---
##  Using `synchronized`

```java
synchronized(this) {
    count++;
}
```
Ensures:
- Mutual exclusion
- Visibility

---
##  What `synchronized` Does Internally
- Uses monitor lock
- Only one thread enters critical section
- Others go to BLOCKED state

---
# 8. Inter-Thread Communication
Uses:
- `wait()`
- `notify()`
- `notifyAll()`

Rules:
- Must be inside synchronized block
- `wait()` releases lock
- `notify()` wakes one waiting thread

---
# 9. Daemon Threads

A daemon thread:
- Runs in background
- Stops when all user threads finish

Example:
- Garbage Collector
```java
thread.setDaemon(true);
```

---

# 1️0. Common Multithreading Problems

###  1.Race Condition
Two threads modify shared variable.

### 2. Deadlock
Thread A waits for B  
Thread B waits for A

###  3. Starvation
Low priority thread never gets CPU.

### 4. Livelock
Threads active but not progressing.

---
# 1️1. Modern Thread Management (Executor Framework)

Instead of manually creating threads:
```java
ExecutorService executor =
    Executors.newFixedThreadPool(5);

executor.submit(() -> System.out.println("Task"));
```

Why?
- Thread pooling
- Better resource management
- Avoids overhead of creating many threads

---

# 11. Top Tricky Interview Questions

---

###  1. Can we restart a thread?
 No  
Once terminated → cannot start again

---

### 2. Why `wait()` must be inside synchronized block?
Because it releases monitor lock.

---
###  3. Difference between `sleep()` and `wait()`?

|sleep()|wait()|
|---|---|
|Does NOT release lock|Releases lock|
|From Thread class|From Object class|

---
###  4. What happens if two threads call `start()` on same object?
IllegalThreadStateException

---
###  5. Is `volatile` enough for thread safety?
No.  
It ensures visibility, not atomicity.

---
###  6. Can constructor be synchronized?

No — object not fully created yet.


---
If you want, I can now create:

- 🔬 Deep dive into synchronization internals (monitor, biased locking, etc.)
    
- 🧠 Advanced concurrency utilities (Locks, Atomic, ConcurrentHashMap)
    
- ⚡ Memory model & happens-before rule explained visually