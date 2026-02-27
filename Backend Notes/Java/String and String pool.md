
# 1️. What Is a `String` in Java?
`String` in Java is:
- An **immutable**
- **final** class 
- Representing a sequence of characters
- Specially optimized by JVM

```java
public final class String
```

### Key Properties
- Immutable → cannot change after creation
- Final → cannot be subclassed
- Thread-safe (because immutable)
- Frequently used → JVM optimizes heavily

---

# 2️. What Is the String Pool?

The **String Pool** (also called _String Constant Pool_) is a special memory area inside the **Heap** where JVM stores unique string literals.

Its goal:
> Avoid creating duplicate String objects to save memory.

---

##  1. Why It Exists

Since strings are used heavily:
```java
String a = "hello";
String b = "hello";
```
Instead of creating 2 objects, JVM creates only **one object** in the pool.
Both `a` and `b` point to the same memory.

---

## 2. Where Is the String Pool Stored?
### Java 6
- Stored in **PermGen**
- Fixed size
- Caused `OutOfMemoryError: PermGen space`
### Java 7+
- Moved to **Heap memory**
- PermGen removed
- Pool became dynamically resizable
### Java 8+
- PermGen completely removed
- Replaced with **Metaspace**
- String Pool lives in **Heap**
- Fully GC-managed

###  Current Implementation (Modern JVM)
- Stored in **Heap**
- Managed by Garbage Collector
- Implemented using a specialized hash table (`StringTable`)
- Can be tuned with JVM flags:
```    
-XX:StringTableSize    
```


---

# 3️. Different Ways to Create Strings

---

##  1. Using Literal

```java
String s = "test";
```
### What Happens?
- JVM checks String Pool
- If `"test"` exists → reuse
- If not → create in pool

- Goes to String Pool  
- Reused  
- No new heap object outside pool

---

##  2. Using `new`

```java
String s = new String("test");
```

### What Happens?
1. `"test"` → goes to pool (if not already)
2. `new String(...)` → creates a new object in heap

Now we have:

```
Pool:  "test"
Heap:  new String("test")
```

Two different objects.

---

##  3. Using `.intern()`

```java
String s = new String("test").intern();
```
### What `intern()` Does
- Checks if equal string exists in pool
- If yes → returns pooled reference
- If not → adds to pool

So:
```java
String s1 = new String("test");
String s2 = s1.intern();
```

Now:

- `s1` → heap object
- `s2` → pooled object

---

# 4. Compile-Time vs Runtime Concatenation

---

## 1. Compile-Time (Goes to Pool)

```java
String s = "a" + "b";
```

Compiler optimizes to:

```java
String s = "ab";
```

-  Stored in pool  
- No runtime object creation

---

## 2. Runtime (Does NOT Go to Pool Automatically)

```java
String a = "a";
String s = a + "b";
```

Behind the scenes:

```java
new StringBuilder(a).append("b").toString();
```

 - New heap object created for s. 
 - Not added to pool (unless interned)

---

# 6️. What Happens When You "Modify" a String?

Strings are immutable.
Example:

```java
String s = "hello";
s.concat("world");
```

What happens?
- A **new String object** is created
- Original `"hello"` remains unchanged
- If result not assigned → lost

Correct usage:
```java
s = s.concat("world");
```

All modifying methods:
- `concat()`
- `replace()`
- `substring()`
- `toUpperCase()`
- `trim()`

👉 ***Always return NEW objects.***

---

# 7️. Tricky Interview Questions

---

##  1. `==` vs `.equals()`

```java
String a = "test";
String b = "test";
```

`a == b` → ✅ true (same pooled object)

---

```java
String a = new String("test");
String b = new String("test");
```

`a == b` → ❌ false (different heap objects)

---

`.equals()` compares content  
`==` compares references

---

##  2. What prints?

```java
String s1 = "hello";
String s2 = new String("hello");
String s3 = s2.intern();

System.out.println(s1 == s2); // ?
System.out.println(s1 == s3); // ?
```

Answer:

```
false
true
```

---

##  3. How Many Objects?

```java
String s = new String("abc");
```

Answer:
- 1 in pool
- 1 in heap  
    = 2 total (if "abc" wasn’t already in pool)

---

##  4. What prints?

```java
String s1 = "ab";
String s2 = "a" + "b";
System.out.println(s1 == s2);
```

Answer:  
✅ true (compile-time optimization)

---

## ❓ 5. What prints?

```java
String s1 = "a";
String s2 = s1 + "b";
String s3 = "ab";

System.out.println(s2 == s3);
```

Answer:  
❌ false (runtime concatenation), s2 would be created in the heap instead of the string pool.

---

##  6. Can String Pool Cause Memory Issues?

In Java 6 → Yes (PermGen overflow)
In Java 7+ → Less likely (Heap + GC managed)
But excessive use of `intern()` on large dynamic data can still cause memory pressure.

---

## 7. Is String Pool Thread-Safe?

Yes.
`String.intern()` uses a synchronized structure internally.

---

##  8. Why Is String Immutable?
1. Security (used in class loading, file paths, URLs)
2. Thread-safety
3. Hashcode caching
4. Enables String Pool

---

# 8️. Internal Implementation (Modern JVM Insight)

Modern `String` (Java 9+):
- Uses `byte[]` instead of `char[]`
- Has a `coder` flag:
    - LATIN1 (1 byte per char)
    - UTF16 (2 bytes per char)

This is called:
> Compact Strings Optimization

Improves memory efficiency significantly.


---

# 🔥 Final Core Takeaways

1. String Pool stores unique literals
2. `new` always forces heap object
3. `intern()` forces pooled reference
4. Compile-time concatenation → pool
5. Runtime concatenation → heap
6. Pool moved from PermGen → Heap in Java 7
7. Modern JVM uses compact strings
8. Immutability enables pooling


#  String vs StringBuilder vs StringBuffer — Quick Overview

| Feature                           | **String**                      | **StringBuilder**               | **StringBuffer**                                                         |     |
| --------------------------------- | ------------------------------- | ------------------------------- | ------------------------------------------------------------------------ | --- |
| Immutability                      | Immutable                       | Mutable                         | Mutable                                                                  |     |
| Thread Safety                     | ✅ Yes (immutable)               | ❌ No                            | ✅ Yes, All the methods use Synchronized keyword, so they are thread safe |     |
| Synchronization                   | Not needed                      | No                              | Yes                                                                      |     |
| Performance (Frequent Changes)    | Slow                            | Fastest                         | Slower than StringBuilder                                                |     |
| Memory Efficiency (Modifications) | Poor                            | Good                            | Good                                                                     |     |
| Uses String Pool                  | Yes (literals only)             | No                              | No                                                                       |     |
| Introduced In                     | Java 1.0                        | Java 5                          | Java 1.0                                                                 |     |
| Typical Use Case                  | Constants, keys, read-only data | Single-threaded string building | Multi-threaded string building                                           |     |
