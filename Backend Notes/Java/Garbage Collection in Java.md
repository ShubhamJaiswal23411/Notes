

# Garbage Collection in Java — Obsidian Notes

---
##  1. What is Garbage Collection (GC)?
###  Definition
Garbage Collection is the **automatic memory management mechanism** in Java that
- Identifies unused objects
- Frees heap memory
- Prevents memory leaks

> Objects that are **no longer reachable** are eligible for GC.

###  Important Clarification
- GC **does not delete objects immediately**
- It runs **periodically / when needed**

---
##  2. What is “Garbage”?

###  Object becomes garbage when:
- No active references point to it

```java
Object obj = new Object();
obj = null; // eligible for GC
```

Java uses **reachability analysis**, not reference counting.
###  Reachability Graph

GC starts from:
- Stack variables
- Static variables
- JNI references
If object is **not reachable → garbage**

---
##  3. Java Memory Model (Heap Structure)

###  Heap Generations
```id="7i6g0s"
Young Generation → Old Generation → Metaspace
```

---
###  Young Generation
- Eden + Survivor spaces (S0, S1)
- New objects are created here
###  Old Generation
- Long-lived objects move here
###  Metaspace (Java 8+)
- Stores class metadata (not in heap)

---

##  4. Types of Garbage Collection Events

###  Minor GC (Young GC)
- Cleans **young generation**
- Fast and frequent
###  Major GC / Full GC
- Cleans **entire heap**
- Slower, causes pauses

---
###  Stop-The-World (STW)
- Application pauses during GC
- Key performance concern

---
##  5. How GC Works (Step-by-Step)

---
### 1️ Mark Phase
- Identify reachable objects
### 2️ Sweep Phase
- Remove unreachable objects
### 3️ Compact Phase
- Remove fragmentation
- Move objects together

---
##  6. Core GC Algorithms (Strategies)

---

##  1. Mark-Sweep
### Steps:
- Mark live objects
- Sweep unused ones
###  Problem:
- Memory fragmentation

---
##  2. Mark-Compact

###  Fix:
- Compacts memory after marking
### Tradeoff:
- More CPU overhead

---
##  3. Copying Algorithm

###  Used in Young Gen
- Copy live objects to another space
###  Tradeoff:
- Needs extra memory

---
##  4. Generational GC (Most Important 🔥)
###  Idea:
- Most objects die young
###  Approach:
- Separate young & old generation
- Optimize differently

---
##  7. Garbage Collectors in Java (Evolution)

---
##  1. Serial GC
- Single-threaded
- Stop-the-world
###  Best for:
- Small apps

---
##  2. Parallel GC (Throughput GC)
- Multi-threaded
- Focus: **high throughput**
###  Downside:
- Long pause times

---
##  3. CMS (Concurrent Mark Sweep) - Deprecated now

###  Feature:
- Concurrent with application
###  Problems:
- Fragmentation
- Complex tuning

---
##  4. G1 GC (Garbage First) 

### Introduced:
Java 7 (default from Java 9)
###  Key Idea:
- Heap divided into **regions**
- Collects regions with most garbage first
###  Benefits:
- Predictable pause times
- Better memory utilization

---
##  5. ZGC (Modern)
###  Features:
- Ultra-low latency (<10ms pauses)
- Concurrent GC
### Key Innovation:
- Colored pointers
- Load barriers
###  Best for:
- Large heaps (GBs to TBs)

---
##  6. Shenandoah GC 
###  Features:
- Low pause times
- Concurrent compaction
###  Used by:
- Red Hat JVMs

---
##  8. GC Evolution Timeline

---

| Java Version | GC                   |
| ------------ | -------------------- |
| Java 5       | Serial, Parallel     |
| Java 6       | CMS                  |
| Java 7       | G1 introduced        |
| Java 8       | CMS popular          |
| Java 9+      | G1 default           |
| Java 11+     | ZGC, Shenandoah      |
| Java 17+     | ZGC production-ready |

---
##  9. Which GC is Best?

###  Depends on Use Case

| Use Case          | Best GC  |
| ----------------- | -------- |
| Small app         | Serial   |
| High throughput   | Parallel |
| Low latency       | G1 / ZGC |
| Ultra-low latency | ZGC      |

---

###  Industry Default Today:

👉 **G1 GC** (balanced)  
👉 **ZGC** (modern, low latency systems)

---
##  10. Important Internal Concepts

---
###  Write Barrier
- Tracks object reference changes
###  Read Barrier (ZGC)
- Ensures correct object access
###  Remembered Sets
- Track cross-region references (G1)

---
##  11. GC Tuning Parameters

---
### Common Flags:

```bash
-XX:+UseG1GC
-XX:+UseZGC
-Xms2g
-Xmx2g
```

---
### G1 Tuning:

```bash
-XX:MaxGCPauseMillis=200
```

---
##  12. Common GC Problems

---
###  Memory Leak
- Objects still referenced unintentionally
###  GC Thrashing
- GC runs too frequently
###  Long Pause Times
- STW impact
###  Fragmentation
- Mostly in CMS

---
##  13. Real-World Example

---

###  E-commerce App
- Many short-lived objects → Young GC heavy
- Large cache → Old Gen pressure

👉 Solution:
- Use G1 or ZGC

---

##  14. Best Practices

---
- Avoid unnecessary object creation
- Use object pooling carefully
- Monitor GC logs
- Choose GC based on workload
- Tune heap size properly

---

#  Tricky Interview Questions 

---

## 1. Why does Java use reachability instead of reference counting?
Because **reference counting cannot handle cyclic references**, while reachability can.
###  Reachability Approach (Java)
Java starts from **GC Roots**:
- Stack variables
- Static fields
- JNI references
👉 If an object is **not reachable from roots**, it is garbage.

---
## 2. What is Stop-The-World (STW) and why is it unavoidable?
STW is when **all application threads pause** so GC can safely operate.
### Why Needed?
GC modifies memory:
- Moves objects
- Updates references
- Cleans memory
👉 If app threads run simultaneously:
- They may access invalid/moved objects 
- Leads to corruption
###  Even modern GCs?
Yes — even **ZGC / G1** have _very short_ STW phases.

---
## 3.  Explain how G1 divides heap into regions
G1 splits heap into **equal-sized regions** instead of fixed generations.
###  Structure
```text
Heap → multiple regions (1MB–32MB each)
```

Each region can be:
- Eden
- Survivor
- Old
###  Working
- Tracks garbage % per region
- Picks regions with **most garbage first**
👉 Hence name: _Garbage First_
###  Benefit
- Predictable pause times
- Better memory utilization

---
## 4. What are remembered sets in G1?
They track **references from one region to another**.
###  Why Needed?
Problem:
- Old → Young references exist
- During Young GC, we must track these
###  Solution: Remembered Set
Each region maintains:
```text
"Who is pointing to me?"
```
###  Benefit
- Avoid scanning entire heap
- Faster GC
###  Tradeoff
- Extra memory overhead
- CPU cost to maintain

---
## 5. How does ZGC achieve low latency?
By doing **most work concurrently** with application threads.
###  Key Techniques
### 1️. Concurrent Phases
- Marking
- Relocation  
	- Happens while app runs
### 2️. Colored Pointers (very important )
- Store metadata in pointers
- Track object state without stopping app
### 3️. Load Barriers
- Fix references _on-the-fly_
###  Result
- Pause times < 10ms
- Even for huge heaps (TBs)
---
## 6. What are colored pointers in ZGC?
Pointers that contain **extra metadata bits** about object state.
###  What they store:
- Marked or not
- Relocated or not
- Remapped or not
###  Why powerful?
Instead of:
- Stopping app → updating all references 
ZGC:
- Updates lazily when accessed 

>Which Enables concurrent GC without long pauses

---
## 7. Difference between write barrier and read barrier?
###  Write Barrier
### Trigger:
When object reference is **modified**
```java
obj.field = newObj;
```
###  Purpose:
- Track changes
- Maintain consistency
###  Read Barrier
###  Trigger:
When object is **accessed**
```java
obj.field
```

###  Purpose:
- Ensure object is in correct state
- Fix references dynamically (ZGC)

---
###  Key Difference

| Feature | Write Barrier | Read Barrier   |
| ------- | ------------- | -------------- |
| Trigger | Write         | Read           |
| Used in | G1, CMS       | ZGC            |
| Purpose | Track updates | Fix references |

---
## 8. High CPU usage due to GC — how do you debug?

### Step-by-step approach

### 1️. Enable GC logs
```bash
-XX:+PrintGCDetails
-XX:+PrintGCDateStamps
```
### 2️. Check:
- GC frequency
- Pause times
- Heap usage
### 3️. Common causes

| Cause            | Fix             |
| ---------------- | --------------- |
| Small heap       | Increase Xmx    |
| Too many objects | Optimize code   |
| Wrong GC         | Switch (G1/ZGC) |
| Memory leak      | Heap dump       |
### 4️. Tools
- VisualVM
- JProfiler
- GC logs analyzer
---
## 9. Application has low latency requirement — which GC?

###  Answer
 **ZGC (best)**  
 **Shenandoah (alternative)**  
 **G1 (if moderate latency ok)**
###  Why ZGC?
- <10ms pauses
- Concurrent processing
- Scales with large heaps

---
## 10. Frequent Full GC — what could be wrong?

###  Possible Reasons
###  1. Heap too small
- Old gen fills quickly
###  2. Memory leak
- Objects never released
###  3. Wrong GC tuning
- Poor parameters
###  4. Excessive promotion
- Too many objects moving to old gen
###  5. Large objects
- Directly allocated in old gen
###  6. Metaspace issues
- Class metadata overflow
###  Debug Approach
- Check GC logs
- Heap dump analysis
- Monitor object allocation

> Full GC = last resort → indicates deeper issue

---
## **11. Latency spikes every few seconds**

### Symptom
- API latency jumps (e.g., 50ms → 2s)
- Happens periodically
### Root Cause
- **Stop-The-World pauses (Minor/Full GC)**
###  How to Debug
- Check GC logs for:
    - Pause time
    - Frequency
- Use tools:
    - VisualVM / JFR
### Fix
- Switch to **G1 or ZGC**
- Tune:
```bash
-XX:MaxGCPauseMillis=100
```

> Latency spikes often correlate directly with GC pause times.

---
## **12. High CPU usage but low throughput**
###  Symptom
- CPU ~90–100%
- App still slow
###  Root Cause
- GC running too frequently (**GC thrashing**)
###  Debug
- Check:
    - GC frequency
    - Allocation rate
###  Fix
- Increase heap:
```bash
-Xms4g -Xmx4g
```

- Reduce object creation
- Optimize data structures

> Too many short-lived objects → frequent Young GC → CPU burn

---
## **13. Frequent Full GC**

### Symptom
- Logs show repeated Full GC
- Long pauses
###  Root Causes
- Old Gen filling up
- Memory leak
- Poor GC config
###  Debug
- Heap dump analysis
- Check object retention
###  Fix
- Increase Old Gen
- Fix leaks
- Use **G1/ZGC**

> Full GC = system under stress

---
## **14. Memory usage keeps increasing (OOM crash)**

### Symptom
- Heap grows continuously
- Ends in OutOfMemoryError
###  Root Cause
- **Memory leak**
### Debug
- Take heap dump:
```bash
-XX:+HeapDumpOnOutOfMemoryError
```
- Analyze with MAT
###  Fix
- Remove static references
- Fix caches / collections

> GC works fine — but objects are still reachable

---
## **15. GC pauses fine, but response still slow**

###  Symptom
- GC logs look normal
- App still slow
### Root Cause
- Not GC-related:
    - DB latency
    - Network calls
    - Thread contention

> Don’t blame GC blindly — validate with data

---
## **16. Large heap but still frequent GC**

###  Symptom
- Heap = large (e.g., 16GB)
- Still frequent GC
###  Root Cause
- High allocation rate
### Debug
- Object creation profiling
### Fix
- Reduce allocations
- Reuse objects

> GC pressure = allocation rate, not just heap size

---
## **17. Microservices app with unpredictable latency**

###  Symptom
- Random spikes in response times
###  Root Cause
- Different services using different GCs
- Some using **Parallel GC (bad for latency)**
### Fix
- Standardize:
    - Use G1 or ZGC across services

> In distributed systems, one slow service affects all

---
## **18. Containerized app behaving differently**

###  Symptom
- Works fine locally
- Poor performance in Docker/K8s
###  Root Cause
- JVM not aware of container limits (older Java)
- Wrong heap sizing
### Fix
```bash
-XX:+UseContainerSupport
-XX:MaxRAMPercentage=75
```

> JVM needs tuning for containers

---
## **19. Choosing the Right GC**

|Scenario|GC|
|---|---|
|Small apps|Serial|
|Throughput-heavy|Parallel|
|Balanced|G1|
|Low latency|ZGC|

---

## **20. Must-Know JVM Flags**

### Heap Size
```bash
-Xms4g   # initial heap
-Xmx4g   # max heap
```
### Choose GC
```bash
-XX:+UseG1GC
-XX:+UseZGC
```
###  Pause Time Goal (G1)
```bash
-XX:MaxGCPauseMillis=200
```
###  GC Logging
```bash
-Xlog:gc*
```
### Heap Dump
```bash
-XX:+HeapDumpOnOutOfMemoryError
```

---
## **21. G1 Tuning Basics**
### Control pause time
```bash
-XX:MaxGCPauseMillis=100
```
###  Region size (rarely tuned)
```bash
-XX:G1HeapRegionSize=16m
```
###  Initiating GC earlier
```bash
-XX:InitiatingHeapOccupancyPercent=45
```

---

## **22. ZGC Tuning Basics**

### Minimal tuning needed:
```bash
-XX:+UseZGC
```
### Optional:
```bash
-Xms = Xmx  # keep equal for stability
```

> ZGC is mostly self-tuning

---
## **23. Golden Architecture Answer**

If asked:
👉 _“How do you design a system to minimize GC issues?”_
### Say:
- Reduce object creation
- Use efficient data structures
- Choose correct GC (ZGC for low latency)
- Tune heap size
- Monitor GC logs
- Avoid large object allocation spikes

