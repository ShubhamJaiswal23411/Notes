
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
## 🔹 Why Multithreading?
- Better CPU utilization
- Improved responsiveness (UI apps)
- Parallel task execution
- Background processing
- High-performance servers

---

## 🔹 Concurrency vs Parallelism

| Term        | Meaning                                     |
| ----------- | ------------------------------------------- |
| Concurrency | Multiple tasks making progress              |
| Parallelism | Tasks executing simultaneously (multi-core) |

Java supports both.

---

# 3️⃣ Thread Creation in Java

There are 3 main ways:

---

## ✅ 1. Extending `Thread` Class

```java
class MyThread extends Thread {
    public void run() {
        System.out.println("Thread running");
    }
}

MyThread t = new MyThread();
t.start();
```

⚠ Always call `start()`, not `run()`.

---

## ✅ 2. Implementing `Runnable` (Recommended)

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

## ✅ 3. Using Lambda (Java 8+)

```java
Thread t = new Thread(() -> {
    System.out.println("Running");
});
t.start();
```

---

# 4️⃣ Thread vs Runnable (Important Interview Topic)

|Feature|Thread|Runnable|
|---|---|---|
|Type|Class|Interface|
|Inheritance|Must extend Thread|Can implement multiple interfaces|
|Reusability|Less flexible|More flexible|
|Best Practice|❌ Not preferred|✅ Preferred|

### Key Insight:

> `Thread` defines execution mechanism  
> `Runnable` defines the task

---

# 5️⃣ Thread Lifecycle

A thread goes through these states:

```
NEW
RUNNABLE
BLOCKED
WAITING
TIMED_WAITING
TERMINATED
```

---

## 🔹 1. NEW

- Thread object created
    
- `start()` not called
    

---

## 🔹 2. RUNNABLE

- `start()` called
    
- Ready to run
    
- Waiting for CPU
    

---

## 🔹 3. BLOCKED

- Waiting for monitor lock (synchronized block)
    

---

## 🔹 4. WAITING

- Waiting indefinitely
    
- Methods:
    
    - `wait()`
        
    - `join()`
        
    - `LockSupport.park()`
        

---

## 🔹 5. TIMED_WAITING

- Waiting for specific time
    
- Methods:
    
    - `sleep()`
        
    - `wait(timeout)`
        
    - `join(timeout)`
        

---

## 🔹 6. TERMINATED

- `run()` method completed
    
- Cannot restart thread
    

---

# 6️⃣ Important Thread Methods

### 🔹 `start()`

- Creates new call stack
    
- Moves thread to RUNNABLE
    

---

### 🔹 `run()`

- Contains task logic
    
- Calling directly = normal method call (no new thread)
    

---

### 🔹 `sleep(milliseconds)`

- Pauses thread
    
- Does NOT release lock
    

---

### 🔹 `join()`

- Waits for another thread to finish
    

---

### 🔹 `yield()`

- Suggests scheduler to give chance to other threads
    

---

### 🔹 `interrupt()`

- Signals thread to stop
    
- Sets interrupt flag
    

---

### 🔹 `isAlive()`

- Checks if thread is still running
    

---

# 7️⃣ Thread Scheduling

- Controlled by JVM + OS
    
- Uses **priority system**
    

```java
thread.setPriority(Thread.MAX_PRIORITY);
```

Priorities:

- 1 (MIN)
    
- 5 (NORM)
    
- 10 (MAX)
    

⚠ Not guaranteed behavior (OS dependent)

---

# 8️⃣ Synchronization & Thread Safety

When multiple threads access shared data → race conditions may occur.

Example:

```java
count++;
```

Not atomic.

---

## 🔹 Using `synchronized`

```java
synchronized(this) {
    count++;
}
```

Ensures:

- Mutual exclusion
    
- Visibility
    

---

## 🔹 What `synchronized` Does Internally

- Uses monitor lock
    
- Only one thread enters critical section
    
- Others go to BLOCKED state
    

---

# 9️⃣ Inter-Thread Communication

Uses:

- `wait()`
    
- `notify()`
    
- `notifyAll()`
    

Rules:

- Must be inside synchronized block
    
- `wait()` releases lock
    
- `notify()` wakes one waiting thread
    

---

# 🔟 Daemon Threads

A daemon thread:

- Runs in background
    
- Stops when all user threads finish
    

Example:

- Garbage Collector
    

```java
thread.setDaemon(true);
```

---

# 1️⃣1️⃣ Common Multithreading Problems

### 🔹 Race Condition

Two threads modify shared variable.

---

### 🔹 Deadlock

Thread A waits for B  
Thread B waits for A

---

### 🔹 Starvation

Low priority thread never gets CPU.

---

### 🔹 Livelock

Threads active but not progressing.

---

# 1️⃣2️⃣ Modern Thread Management (Executor Framework)

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

# 🎯 Top Tricky Interview Questions

---

### ❓ 1. Difference between `start()` and `run()`?

`start()` → creates new thread  
`run()` → normal method call

---

### ❓ 2. Can we restart a thread?

❌ No  
Once terminated → cannot start again

---

### ❓ 3. Why `wait()` must be inside synchronized block?

Because it releases monitor lock.

---

### ❓ 4. Difference between `sleep()` and `wait()`?

|sleep()|wait()|
|---|---|
|Does NOT release lock|Releases lock|
|From Thread class|From Object class|

---

### ❓ 5. What happens if two threads call `start()` on same object?

IllegalThreadStateException

---

### ❓ 6. Is `volatile` enough for thread safety?

No.  
It ensures visibility, not atomicity.

---

### ❓ 7. What is context switching?

CPU switching between threads.

---

### ❓ 8. Why is `Runnable` preferred over `Thread`?

Better design, flexibility, and separation of concern.

---

### ❓ 9. Can constructor be synchronized?

No — object not fully created yet.

---

### ❓ 10. What is thread-safe class?

Class safe for concurrent access without data corruption.

---

# 🚀 Final Summary

- Thread = lightweight unit of execution
    
- Multithreading improves performance
    
- Preferred way → implement `Runnable`
    
- Lifecycle: NEW → RUNNABLE → BLOCKED/WAITING → TERMINATED
    
- Use synchronization to avoid race conditions
    
- Modern approach → Executor framework
    
- Understand lifecycle + synchronization deeply for interviews
    

---

If you want, I can now create:

- 🔬 Deep dive into synchronization internals (monitor, biased locking, etc.)
    
- 🧠 Advanced concurrency utilities (Locks, Atomic, ConcurrentHashMap)
    
- ⚡ Memory model & happens-before rule explained visually