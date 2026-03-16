
## 1️. What is Collection Framework?

The **Java Collection Framework (JCF)** is a unified architecture for storing and manipulating groups of objects.

Introduced in **Java 2 Platform SE 1.2 API Specification**, it provides:
- Interfaces (abstract data types)
- Implementations (concrete classes)
- Algorithms (static utility methods in **Oracle Corporation**'s `Collections` class)

---
#  High-Level Hierarchy

```
Iterable (interface)
   └── Collection (interface)
         ├── List
         ├── Set
         └── Queue
              └── Deque

Map (separate hierarchy)
```

 `Map` does NOT extend `Collection`.


![[Collection Hierarchy.png]]

---
# 2️. Iterable Interface

### Purpose
Root interface allowing objects to be used in enhanced for-loop.

### Key Method
```java
Iterator<T> iterator();
```

---
# 3️. Collection Interface

Defines common operations for all collections.

### Common Methods (Keep short as requested)

| Method             | Input   | Output   |
| ------------------ | ------- | -------- |
| add(E e)           | element | boolean  |
| remove(Object o)   | object  | boolean  |
| size()             | -       | int      |
| isEmpty()          | -       | boolean  |
| contains(Object o) | object  | boolean  |
| clear()            | -       | void     |
| iterator()         | -       | Iterator |

---

# 4️. LIST Hierarchy

## Characteristics
- Ordered
- Allows duplicates
- Index-based access

---
## 4.1 ArrayList

### Structure
Dynamic array implementation.
### Internal Working
- Backed by **Object[]**
- Default capacity = 10
- Growth formula (Java 8+):

```
newCapacity = oldCapacity + (oldCapacity >> 1)
            = oldCapacity * 1.5
```

So capacity increases by **50%**

### Time Complexity

| Operation  | Time           |
| ---------- | -------------- |
| get()      | O(1)           |
| add()      | O(1) amortized |
| add(index) | O(n)           |
| remove()   | O(n)           |
### When to Use
- Frequent reads
- Less insert/delete in middle

---
## 4.2 LinkedList

### Structure
Doubly linked list.

### Internal Working
Each node contains:
```
prev | data | next
```

Maintains:
- head
- tail

### Time Complexity

| Operation        | Time                 |
| ---------------- | -------------------- |
| get(index)       | O(n)                 |
| addFirst/addLast | O(1)                 |
| remove           | O(1) (if node known) |
### When to Use
- Frequent insertion/deletion
- Implementing Queue/Deque

---
## 4.3 Vector (Legacy)
- Synchronized
- Growth: doubles capacity (100%)
### Why Avoid?
Replaced by `ArrayList`.

---
## 4.4 Stack (Legacy)
Extends Vector.
Use `Deque` instead.

---

## 4.5 CopyOnWriteArrayList (Introduced in java 5)

### Package
```
java.util.concurrent
```
Part of the concurrent collections introduced in **Oracle Corporation**'s concurrency utilities (Java 5).

---

###  What is CopyOnWriteArrayList?
A **thread-safe variant of ArrayList** where:
> Every write operation creates a **new copy of the underlying array**.

It is designed for:
- Read-heavy
- Write-rare
- Multi-threaded environments

---
###  Internal Working
### Data Structure Used

```
private transient volatile Object[] array;
```

Key points:
- Backed by **Object[]** (just like ArrayList)
- Marked **volatile** → guarantees visibility across threads
- Uses **ReentrantLock** during write operations

---
###  Write Operation (Core Concept)
When you do:
```java
list.add(e);
```

Internally:
1. Acquire lock
2. Copy entire array
3. Add new element
4. Replace old array reference
5. Release lock

Pseudo:
```java
Object[] oldArray = array;
Object[] newArray = Arrays.copyOf(oldArray, oldArray.length + 1);
newArray[last] = e;
array = newArray;
```

⚠ Entire array is copied on every modification.

---
### Growth Behavior
Unlike ArrayList:
- No incremental resizing logic like 1.5x
- Every modification creates:
    ```
    new array of size = old + 1 (or adjusted size)
    ```

So:
- No capacity buffer
- No load factor
- No resizing threshold

Memory cost is high during writes.

---
### Thread Safety Mechanism
### Reads:
- No lock required
- Direct access to volatile array
- Very fast
### Writes:
- Uses **ReentrantLock**
- Ensures atomic replacement of array

---
### Iterator Behavior (Very Important)

### Snapshot Iterator (Fail-Safe)
When iterator is created:
```java
Iterator<E> it = list.iterator();
```

It captures current array reference.
Even if list changes later:
- Iterator works on old snapshot
- No ConcurrentModificationException

---
### Time Complexity

| Operation  | Time |
| ---------- | ---- |
| get()      | O(1) |
| add()      | O(n) |
| remove()   | O(n) |
| contains() | O(n) |
| iteration  | O(n) |

⚠ Writes are expensive.

---
### When to Use
 Ideal for:
- Event listeners
- Observer pattern
- Configuration lists
- Read-heavy systems
- Rare modification scenarios

Example:
- Logging subscribers
- Web server request filters

---
###  When NOT to Use

 Frequent writes  
 Large datasets with frequent updates  
 Memory-sensitive systems


---
# 5️. SET Hierarchy

##  Characteristics
- No duplicates
- Based on `equals()` and `hashCode()`

---
## 5.1 HashSet
### Internal Working
Backed by **HashMap**
Actually:
```java
HashMap<E, Object>
```

Dummy value = `PRESENT`
### Structure
- Array of buckets
- Each bucket:
    - Linked List (Java 7)
    - Linked List + Red-Black Tree (Java 8+)

### Tree Conversion Rules (Java 8+)
- If bucket size > 8 → convert to Red-Black Tree
- If size < 6 → convert back to LinkedList
- Minimum capacity must be ≥ 64
### Time Complexity
O(1) average

---
## 5.2 LinkedHashSet
- Extends HashSet
- Maintains insertion order
- Uses doubly linked list internally

---
## 5.3 TreeSet
Backed by:
```java
TreeMap
```

Implements:
- Red-Black Tree

Time Complexity:  
O(log n)

Requires:
- Comparable OR Comparator

---
# 6️. QUEUE Hierarchy

## 6.1 PriorityQueue
### Internal Working
- Binary Heap
- Backed by array
Min-heap by default.
### Time Complexity

| Operation | Time     |
| --------- | -------- |
| add       | O(log n) |
| poll      | O(log n) |
| peek      | O(1)     |

---
## 6.2 ArrayDeque
- Resizable circular array
- Faster than Stack & LinkedList
Growth: doubles capacity

---
# 7️. MAP Hierarchy (Separate)

## 7.1 HashMap
### Internal Structure

```
Node<K,V>[] table
```

Each bucket:
- LinkedList
- Red-Black Tree (if >8 nodes)
### Default Values

| Property         | Value                 |
| ---------------- | --------------------- |
| Default capacity | 16                    |
| Load factor      | 0.75                  |
| Resize threshold | capacity * loadFactor |

### Resize Rule
Capacity doubles (×2)
### Hashing
```java
hash = key.hashCode()
hash ^= (hash >>> 16)
```

Index:
```
(n - 1) & hash
```

### Time Complexity
O(1) average  
O(log n) (tree bucket)

---
## 7.2 LinkedHashMap

Maintains:
- Insertion order
- Access order (if configured)

Uses:  
Doubly linked list + HashMap

---
## 7.3 TreeMap
- Red-Black Tree
- Sorted keys
- O(log n)

---
## 7.4 Hashtable (Legacy)
- Synchronized
- No null keys
- Slower

---
# 8️. Red-Black Tree (Used in TreeMap, TreeSet, HashMap buckets)

Self-balancing BST.
Properties:
1. Node is red or black
2. Root is black
3. No two consecutive red nodes
4. Same black height

Height ≈ log n

---
# 9️. Fail-Fast vs Fail-Safe

### Fail-Fast
- ArrayList
- HashMap
- HashSet

Uses `modCount`

Throws:
```
ConcurrentModificationException
```

---

### Fail-Safe
- ConcurrentHashMap
- CopyOnWriteArrayList

Work on copy.

---
# 10. Concurrent Collections

## ConcurrentHashMap

### Java 7
Segment-based locking.

### Java 8
- No segments
- CAS + synchronized on bucket
- Tree bins supported

Allows:
- Concurrent read & write

---
# 1️1️. Equality & Hashing Contract

If:
```
a.equals(b) == true
```

Then:
```
a.hashCode() == b.hashCode()
```

Violating this breaks HashSet/HashMap.

---
# 1️2️. Important Algorithms (Collections Class)

From **Oracle Corporation**
- sort()
- reverse()
- shuffle()
- binarySearch()
- min()
- max()
- frequency()

---
#  Internal Growth Summary

| Class         | Initial Capacity | Growth |
| ------------- | ---------------- | ------ |
| ArrayList     | 10               | 1.5x   |
| Vector        | 10               | 2x     |
| HashMap       | 16               | 2x     |
| ArrayDeque    | 16               | 2x     |
| PriorityQueue | 11               | 1.5x   |

---
#  TRICKY INTERVIEW QUESTIONS (Advanced)

---
### 1️. Why HashMap capacity is always power of 2?

To use:
```
(n - 1) & hash
```
Instead of modulus → faster.

---
### 2️. Why load factor 0.75?
Tradeoff:
- Space vs Collision
- Empirically optimal

---
### 3️. What happens if hashCode() always returns 1?
All elements go into one bucket.
Java 8 → converts to Red-Black Tree after 8 entries.
Time becomes O(log n)

---
### 4️. Why String is good key?
- Immutable
- Cached hashCode
- Final class

---
### 5️. Difference between HashMap and ConcurrentHashMap?
- HashMap allows 1 null key
- ConcurrentHashMap allows none
- Different locking mechanism

---
### 6️. Why ConcurrentHashMap doesn’t allow null?
To avoid ambiguity:
```
map.get(key) == null
```

Is it absent or mapped to null?

---
### 7️. What is structural modification?
Add/remove elements → increments modCount.

---
### 8️. Can we override equals without hashCode?
Yes, but breaks HashMap.

---
### 9️. Why TreeSet cannot store heterogeneous elements?
Because comparison required.

---
### 10. Difference between remove() and poll() in Queue?
- remove() throws exception if empty
- poll() returns null

---
### 1️1️. Why ArrayList remove is O(n)?
Because of shifting.

---
### 1️2️. Why LinkedList get() is slow?
No index access → traversal required.

---
### 1️3️. What is IdentityHashMap?
Uses:
```
== instead of equals()
```

---
### 1️4️. Why HashMap not thread safe?
Resize operation can cause infinite loop (Java 7 issue).

---
### 1️5️. Explain HashMap resize in detail.
- Capacity doubles
- Rehash NOT required fully
- Old bucket entries redistributed using:

```
hash & oldCap
```

Optimization in Java 8.

---
### 1️6️. What is weak reference in WeakHashMap?
Entries removed when key no longer strongly referenced.

---
### 1️7️. Why CopyOnWriteArrayList good for read-heavy?
Because:
- Writes create new copy
- Reads lock-free

---
### 1️8️. Difference between offer() and add() in Queue?
add() throws exception  
offer() returns false

---
### 1️9️ What is NavigableMap?
Provides:
- lowerKey()
- higherKey()
- ceilingKey()
- floorKey()

Implemented by TreeMap.

---
### 2️0️. How does PriorityQueue maintain heap?
Binary heap:  
Parent index = (i - 1)/2  
Left = 2i + 1  
Right = 2i + 2

---

#  Interview Deep-Dive Scenarios
1. Design LRU Cache → LinkedHashMap accessOrder=true
2. Count duplicates efficiently → HashMap
3. Thread-safe high concurrency → ConcurrentHashMap
4. Sorted data with frequent insert → TreeSet
5. FIFO → ArrayDeque
6. Stack → ArrayDeque

---
#  Ultimate Interview Killer Questions
- How does HashMap avoid full rehash during resize?
- Explain treeification threshold and why 8?
- Why threshold to untreeify is 6?
- What is alternative hashing?
- How fail-fast implemented internally?
- What happens if two threads resize HashMap simultaneously?
- Why ConcurrentHashMap size() is approximate?
- Difference between synchronizedMap and ConcurrentHashMap?
- Explain memory layout of HashMap Node.
- What is difference between Comparable and Comparator in TreeMap?
- Can we change comparator after TreeSet creation?
- How does LinkedHashMap maintain order?
- Why HashMap iteration order appears random?
- What happens if mutable key changes after insertion?

---

# 🧠 Final Master Tip

If you deeply understand:
- Hashing
- Resizing
- Treeification
- Red-Black trees
- Load factor logic
- modCount mechanism

You can crack **any Java collections interview**.
