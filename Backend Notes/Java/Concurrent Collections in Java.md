
#  1. Java Concurrent Collections (java.util.concurrent)

---
##  Key Design Goals
- Thread safety
- High concurrency
- Non-blocking / minimal blocking
- Scalability

---
#  2. Concurrent Alternatives Overview

| Normal Collection | Concurrent Alternative                      |
| ----------------- | ------------------------------------------- |
| `ArrayList`       | `CopyOnWriteArrayList`                      |
| `HashSet`         | `CopyOnWriteArraySet`                       |
| `HashMap`         | `ConcurrentHashMap`                         |
| `Queue`           | `ConcurrentLinkedQueue`                     |
| `BlockingQueue`   | `ArrayBlockingQueue`, `LinkedBlockingQueue` |

---

# 3. Locking Mechanisms Overview

|Mechanism|Used In|Description|
|---|---|---|
|Synchronized (Intrinsic Lock)|Old collections|Full lock|
|ReentrantLock|BlockingQueue|Fine-grained control|
|CAS (Compare-And-Swap)|ConcurrentHashMap, Queue|Lock-free|
|Segment Locking (old CHM)|Pre-Java 8|Partitioned locks|
|Node-level locking|Java 8+ CHM|Fine-grained|
|Copy-on-write|COW collections|No lock for reads|

![[Concurrent Collections and their Locking mechanisms.png]]

---

#  4. ConcurrentHashMap (MOST IMPORTANT)

---
## 1. Overview
- Thread-safe alternative to `HashMap`
- High concurrency
##  2. Internal Working (Java 8+)
- Uses:
    - **CAS**
    - **synchronized blocks (on bins)**
    - **Node-level locking**
## 3. Structure

```text
Array of Nodes (buckets)
→ Linked List OR Tree (if collisions high)
```
## 4. Locking Strategy

### Before Java 8:
- Segment-based locking
### Java 8+:
- Lock only **bucket (bin)**
- Uses CAS for insertion
## 5. Key Properties
- No `null` keys/values
- Thread-safe reads without locking
- High throughput
## 6. Why no `null`?
 Ambiguity:
- `get()` returns null → key absent OR value null?

---
#  5. CopyOnWriteArrayList

---
## 1. Idea
 On write:
- Create **new copy**
- Modify copy
## 2. Internal Behavior
- Uses **ReentrantLock** for writes
- Reads are:
    - Lock-free
    - Snapshot-based
## 3. Locking
- Write → Lock
- Read → No lock
## 4. Best Use Case
- Read-heavy systems
- Rare writes
## 5. Problems
- Memory overhead
- Slow writes

---
#  6. ConcurrentLinkedQueue

## 1. Type
- Non-blocking queue
## 2. Internal Working
- Uses **CAS (lock-free)**
- Based on **Michael-Scott algorithm**
## 3. Properties
- No locks
- High scalability
- Eventually consistent size

---
#  7. Blocking Queues

### 1️. ArrayBlockingQueue
- Fixed size
- Uses **ReentrantLock**
### 2️. LinkedBlockingQueue
- Optional bounded
- Separate locks:
    - Put lock
    - Take lock
## 3. Example

```java
BlockingQueue<Integer> queue = new ArrayBlockingQueue<>(10);
queue.put(1);
queue.take();
```
## 4. Key Concept
- Producer-Consumer model

---
#  8. CopyOnWriteArraySet
## 1. Built on:
- `CopyOnWriteArrayList`
## 2. Behavior
- Same copy-on-write strategy
---

# 9. Synchronized vs Concurrent Collections

---

|Feature|Synchronized|Concurrent|
|---|---|---|
|Locking|Whole object|Fine-grained|
|Performance|Low|High|
|Iteration|Fail-fast|Weakly consistent|

---
# 10. Iterators (VERY IMPORTANT)
##  Types

---

### 1️. Fail-Fast (Normal Collections)

```java
list.add(1);
for (Integer i : list) {
    list.add(2); // ❌ ConcurrentModificationException
}
```

---

### 2️. Fail-Safe / Weakly Consistent (Concurrent)

```java
ConcurrentHashMap map = new ConcurrentHashMap();
```
- No exception
- Reflects partial updates

---

# 11. CAS (Compare-And-Swap)

---
## 1. What is CAS?
Atomic operation:
```text
if (current == expected)
    update
```
## 2. Used In
- ConcurrentHashMap
- ConcurrentLinkedQueue
- Atomic classes
## 3. Problems
- ABA problem
- Spin-wait overhead

---
# 12. Tricky Interview Questions

---

## 1. Why ConcurrentHashMap is faster than Hashtable?
- Fine-grained locking vs full lock
## 2. Why ConcurrentHashMap does not allow null?
- Avoid ambiguity in concurrent reads 
## 3. Difference between fail-fast and fail-safe?

|Fail-fast|Fail-safe|
|---|---|
|Throws exception|No exception|
|Uses original collection|Uses snapshot|
## 4. Why CopyOnWrite is fast for reads?
- No locking 
- Immutable snapshot
## 5. What is weakly consistent iterator?
- Reflects some updates but not guaranteed all
## 6. What locking does ConcurrentLinkedQueue use?
- Lock-free (CAS)
## 7. Why BlockingQueue uses two locks?
- Increase concurrency (put & take separate)
## 8. Can ConcurrentHashMap resize safely?
- Yes, using CAS + help from threads
## 9. Why size() is expensive in ConcurrentHashMap?
- Needs traversal due to concurrent updates
## 10. When to use which?

|Use Case|Collection|
|---|---|
|Read-heavy|CopyOnWriteArrayList|
|High concurrency map|ConcurrentHashMap|
|Producer-consumer|BlockingQueue|
|Lock-free queue|ConcurrentLinkedQueue|

---
# 13. Real-World Mapping

---
 Web Servers
- CHM for caching
Messaging Systems
- BlockingQueue
Event Systems
- ConcurrentLinkedQueue
