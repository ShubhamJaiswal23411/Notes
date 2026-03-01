
Below are **in-depth Obsidian-style notes** (structured, interconnected, interview-focused) on **Java locking mechanisms & deadlocks**.

I’ve reordered the flow for better conceptual understanding:

1. Why Locks Are Needed
    
2. Intrinsic Locks (`synchronized` / Monitor Locks)
    
3. Explicit Locks (`Lock` API)
    
4. ReentrantLock
    
5. ReentrantReadWriteLock
    
6. Deadlocks
    
7. Tricky Interview Questions
    

---

#  Concurrency & Locks in Java

---
# 1️. Why Do We Need Manual Locks?

##  1.The Core Problem: Race Conditions

When multiple threads access shared mutable state:
- Operations like `count++` are **not atomic**
- CPU caching & reordering cause visibility issues
- Inconsistent state may be observed
Without synchronization → unpredictable behavior.

---

## 2. Then Why Not Just Use `synchronized`?

Provides synchronization and prevents race condition but it does have other issues like :
- *Blocking* - threads wait indefinetly
- *Fairness* - starvation might happen but in case of reentrant locks we have the option for the lock to be fair
- *Interuptability* - waiting threads cant be interuppted they will wait indefinetly
- *Read Write Locking* - synchonized treats everything as a write operation and blocks all other threads from entering the critical section even if they are just trying to read the data 

---

# 2️. Monitor Locks (`synchronized`)

Every Java object has an **intrinsic monitor lock**.

When a thread enters:
```java
synchronized(obj) {
   // critical section
}
```

It:
1. Acquires the object's monitor
2. Executes code
3. Releases monitor automatically

---

## 1. How It Works Internally

Uses **monitorenter / monitorexit bytecode instructions**.
The JVM manages:
- Lock acquisition
- Lock release
- Wait set (for `wait()`)
- Entry set (blocked threads)

---
##  2. Key Properties

| Feature             | Behavior                      |
| ------------------- | ----------------------------- |
| Reentrant           | Yes                           |
| Fairness            | No                            |
| Interruptible       | No                            |
| Try-lock            | No                            |
| Timed lock          | No                            |
| Condition variables | Only 1 implicit (wait/notify) |
| Auto release        | Yes                           |

---
## 3. Advantages
- Simple
- Automatic unlock (exception safe)
- JVM optimizations (biased locking, lightweight locking)

---

##  4. Limitations
- Cannot interrupt a blocked thread
- Cannot attempt timed lock acquisition
- Only one condition queue per object
- No fairness control

---
# 3️. Manual Locks (`java.util.concurrent.locks`)
Introduced in **Java Concurrency in Practice era (Java 5)**.
The main interface:
```java
Lock lock = new ReentrantLock();
lock.lock();
try {
   // critical section
} finally {
   lock.unlock();
}
```

---
# 4️. Why Manual Locks Over `synchronized`?

## 1. Major Advantages

### 1️. Interruptible Lock Acquisition
```java
lock.lockInterruptibly();
```

Thread can respond to interrupts while waiting.
`synchronized` → cannot do this.

---
### 2️. Try Lock (Non-blocking)

```java
if(lock.tryLock()) {
   ...
}
```

Prevents deadlock-prone designs.

---
### 3️. Timed Lock

```java
lock.tryLock(5, TimeUnit.SECONDS);
```

Avoid indefinite waiting.

---
### 4️. Fairness Policy

```java
new ReentrantLock(true); // fair
```

FIFO ordering.
`synchronized` → no fairness guarantee.

---
### 5️. Multiple Condition Variables

```java
Condition notFull = lock.newCondition();
Condition notEmpty = lock.newCondition();
```

Unlike monitor (single wait set).
Useful for producer-consumer problems.

---
### 6️. Better Scalability Under Contention

Under high contention, explicit locks often outperform monitors.

---

## 2. Trade-off

You must manually release:
```java
finally {
   lock.unlock();
}
```

If forgotten → catastrophic bugs.

---

# 5️. ReentrantLock

Located in:
`java.util.concurrent.locks.ReentrantLock`

---
## 1. What Does "Reentrant" Mean?

A thread can acquire the same lock multiple times.
Internally maintains:
```
owner thread
hold count
```

Example:
```java
lock.lock();
lock.lock(); // allowed
lock.unlock();
lock.unlock();
```

If unlock count < lock count → lock not released.

---
## 2. How It Works Internally
Built on:
- AbstractQueuedSynchronizer (AQS)
- CLH Queue (FIFO wait queue)
- CAS operations (compare-and-swap)

---
## 3. Fair vs Non-Fair

| Type               | Behavior          |
| ------------------ | ----------------- |
| Non-fair (default) | Faster, can barge |
| Fair               | FIFO ordering     |

Fair locks reduce starvation but reduce throughput.

---

# 6️. Monitor Locks vs ReentrantLock

| Feature             | synchronized | ReentrantLock                |
| ------------------- | ------------ | ---------------------------- |
| Reentrant           | ✅            | ✅                            |
| Interruptible       | ❌            | ✅                            |
| Timed               | ❌            | ✅                            |
| Fairness            | ❌            | ✅                            |
| Multiple Conditions | ❌            | ✅                            |
| Performance         | Good         | Better under high contention |
| Auto release        | ✅            | ❌                            |

---
# 7️. ReentrantReadWriteLock

Class:
`java.util.concurrent.locks.ReentrantReadWriteLock`

---

## 1. Why Needed?
When:
- Many readers
- Few writers
- Reads don't modify state
Using a normal lock → read threads block each other unnecessarily.

---
##  Two Locks Inside
1. **Read Lock**
2. **Write Lock**

---
##  Rules
- Multiple readers allowed simultaneously
- Only one writer allowed
- Writer blocks readers
- Readers block writer (depending on fairness)

---
##  Use Case
Cache systems  
Configuration objects  
In-memory data stores

---
##  Internals

Also built on AQS.
Uses state bits:
```
Higher 16 bits → read count
Lower 16 bits → write count
```

---
##  Potential Problem

Writer starvation in non-fair mode.

---  
# 8️.  Necessary Conditions for Deadlock (Coffman Conditions)

All four must exist:
-  Mutual Exclusion - Resource cannot be shared.
- Hold and Wait - Thread holds one resource while waiting for another.
- No Preemption - Resource cannot be forcibly taken.
- Circular Wait
		Thread A → waits for B  
		Thread B → waits for C  
		Thread C → waits for A

Break any one → no deadlock.

---

## 1. How to Prevent Deadlock?

- Lock ordering (always acquire in same order)
- Use tryLock with timeout
- Avoid nested locks
- Use higher-level concurrency utilities

---

# Tricky Interview Questions

---

### 1. Is `synchronized` reentrant? How?
Yes.
Each thread maintains a lock hold count inside monitor.
### 2. Can `ReentrantLock` cause deadlock?
Yes. It doesn’t prevent deadlock automatically.
### 3. What happens if `unlock()` is not called?
Other threads block forever → resource leak → possible deadlock.
### 4. Can two threads hold read lock simultaneously?
Yes.
But not if a writer holds write lock.
### 5. Is `ReentrantReadWriteLock` always better?
No.
If writes are frequent → worse performance.
### 6. Difference Between Monitor Wait Set & Condition?
Monitor:
- Single wait set per object

ReentrantLock:
- Multiple condition queues
### 7. Does fairness guarantee no starvation?
Not strictly — but reduces probability.
### 8. What happens if thread holding lock dies?
- For `synchronized` → JVM releases monitor
- For `ReentrantLock` → lock released only if thread exits normally (no, JVM still releases because ownership tied to thread lifecycle)
### 9. Can deadlock happen with a single thread?
No.
Deadlock requires circular wait involving multiple threads.
### 10. Why is `tryLock()` important in system design?
Helps:
- Avoid deadlock
- Implement backoff strategies
- Build responsive systems

# 🧾 Short Summary

- `synchronized` → simple, JVM-managed monitor lock.
- `ReentrantLock` → advanced features: fairness, interruptible, multiple conditions.
- `ReentrantReadWriteLock` → improves read-heavy concurrency.
- Deadlock requires 4 Coffman conditions.
- Manual locks give flexibility but require discipline.
