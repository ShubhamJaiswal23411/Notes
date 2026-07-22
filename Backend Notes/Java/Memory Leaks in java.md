
##  1. What is a Memory Leak?

###  Definition
A memory leak occurs when:
> Objects are **no longer needed logically** but are still **reachable**, so GC cannot reclaim them.

---
##  2. How Memory Leaks Happen (Core Idea)
### Root Cause
Objects remain **reachable from GC Roots** unintentionally.
###  GC Roots include:
- Static variables
- Thread stacks
- JNI references
 
---
##  3. Common Causes of Memory Leaks

##  1. Static Collections

```java
static List<Object> cache = new ArrayList<>();
```
### Problem
- Static lives for entire app lifecycle
- Objects never removed
---
##  2. Unbounded Caches
```java
Map<String, Object> map = new HashMap<>();
```
### Problem
- Keeps growing forever
### Fix
- Use LRU cache (e.g., LinkedHashMap)
---
##  3. Listener / Callback Leaks

```java
list.addListener(obj);
```
###  Problem
- Listener holds reference → prevents GC

---
##  4. ThreadLocal Leaks

```java
ThreadLocal<MyObj> tl = new ThreadLocal<>();
```
###  Problem
- Values not removed → tied to thread lifecycle
###  Fix
```java
tl.remove();
```

---
##  5. Inner Class / Anonymous Class
- Holds implicit reference to outer class

---
##  6. Improper equals/hashCode
- Prevents removal from collections

---
##  7. ClassLoader Leaks (Advanced )
- Common in app servers
- Classes never unloaded

---
##  8. Large Object Retention
- Big objects kept in memory unnecessarily

---
##  4. Symptoms of Memory Leak

---
###  Observable Behavior

- Increasing heap usage
- Frequent Full GC
- Long GC pauses
- Eventually → `OutOfMemoryError`

---
##  5. How to Detect Memory Leaks

## Step-by-Step Debugging

### 1️. Enable GC logs

```bash
-Xlog:gc*
```

Look for:
- Heap not reducing after GC
- Increasing baseline
### 2️. Monitor Heap Usage
Tools:
- VisualVM
- JConsole
### 3️. Take Heap Dump

```bash
-XX:+HeapDumpOnOutOfMemoryError
```
### 4️. Analyze Heap Dump
Tools:
- Eclipse MAT (Memory Analyzer Tool)
###  What to Look For
- Dominator Tree
- Retained size
- Reference chains to GC roots
###  Key Question

> Why is this object still reachable?

---
##  6. Key Concepts in Heap Analysis

###  Dominator Tree
- Shows which object retains most memory
###  Retained Size
- Memory freed if object is removed
###  GC Roots Path
- Shows why object is not collected

---
##  7. Example Debug Flow

### Scenario:
Heap grows continuously
### Steps:
1. Take heap dump
2. Find largest objects
3. Trace to GC roots
4. Identify holding reference
5. Fix code

---
##  8. Prevention Strategies

###  Best Practices
- Avoid static references
- Use weak references if needed
- Clean ThreadLocal
- Use bounded caches
- Monitor memory regularly

---

# 🎯 Tricky Interview Questions + Answers

---

### 1. Can cyclic references cause memory leaks in Java?
No, Java GC handles cycles via reachability

---

### 2. How do you identify a memory leak in production?
 Steps:
- Monitor heap growth
- Check GC logs
- Take heap dump
- Analyze dominator tree
---
### 3. What is dominator tree?
shows objects holding most memory

---
### 4. Why can static variables cause memory leaks?
 Because they are GC roots → always reachable

---
### 5. How do ThreadLocal leaks happen?
 Values tied to thread lifecycle  
 Not removed → leak

---
### 6. Frequent Full GC but memory not freed?
Likely memory leak

---
### 7. How would you fix a cache-related leak?
 Use:
- LRU cache(Bounded caches)
- WeakHashMap (uses a weak reference mean memory is till garbage collectable)
