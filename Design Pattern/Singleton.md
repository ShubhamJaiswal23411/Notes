
##  1. What is Singleton?
A **Singleton** ensures:
- Only **one instance** of a class exists
- Provides a **global access point** to it
###  Real-world examples
- Logger
- Configuration manager
- DB connection pool
- Caches

---

##  Core Characteristics
- Private constructor
- Static instance
- Public access method

```java
public class Singleton {
    private static Singleton instance = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return instance;
    }
}
```

---

#  2. Types of Singleton Implementations

---

## 1️. Eager Initialization

### Idea
Instance created at **class loading time**
```java
public class Singleton {
    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```
### Pros
- Thread-safe (classloader guarantees it)
- Simple
###  Cons
- Memory waste if unused
- No lazy loading

## 2️. Lazy Initialization (Non-thread-safe)

###  Idea
Create instance **only when needed**
```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
###  Problem
- **Not thread-safe**
- Multiple threads → multiple instances

## 3️. Synchronized Method (Thread-safe but slow)

```java
public class Singleton {
    private static Singleton instance;

    private Singleton() {}

    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
###  Problem
- Performance overhead (lock on every call)
## 4️. Double-Checked Locking (DCL)

###  Optimized thread-safe solution
```java
public class Singleton {
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {                 // First check
            synchronized (Singleton.class) {
                if (instance == null) {         // Second check
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

###  Why `volatile` is REQUIRED?
Without `volatile`, **instruction reordering** can happen:
Steps:
1. Allocate memory
2. Assign reference
3. Initialize object

Reordering:
- Step 2 happens before 3 → another thread gets **partially constructed object**

###  `volatile` prevents:
- Reordering
- Visibility issues

## 5️. Bill Pugh Singleton (BEST for interviews)

###  Uses inner static class
```java
public class Singleton {

    private Singleton() {}

    private static class Holder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return Holder.INSTANCE;
    }
}
```
###  Why it works?
- Class loading is **lazy**
- JVM guarantees **thread safety**
- No synchronization overhead
### Pros
- Lazy + thread-safe
- Best performance
## 6️. Enum Singleton (MOST ROBUST)
```java
public enum Singleton {
    INSTANCE;

    public void doSomething() {
        System.out.println("Doing work...");
    }
}
```
###  Why Enum is special?
- Handles:
    - Serialization
    - Reflection
    - Thread safety
### Internally:
- JVM ensures only one instance

---

#  3. Breaking Singleton (IMPORTANT)

---
## 1️. Reflection Attack

```java
Constructor<Singleton> constructor = Singleton.class.getDeclaredConstructor();
constructor.setAccessible(true);
Singleton s2 = constructor.newInstance();
```
###  Breaks all except:
- Enum Singleton

---
## 2️. Serialization Attack

```java
ObjectOutputStream.writeObject(instance);
ObjectInputStream.readObject();
```
###  Creates new instance

###  Fix
```java
protected Object readResolve() {
    return instance;
}
```

---
## 3️. Cloning Attack

```java
@Override
protected Object clone() throws CloneNotSupportedException {
    throw new CloneNotSupportedException();
}
```

---
#  Comparison Table

| Approach     | Lazy | Thread-safe | Performance | Safe from Reflection | Safe from Serialization |
| ------------ | ---- | ----------- | ----------- | -------------------- | ----------------------- |
| Eager        | ❌    | ✅           | ✅           | ❌                    | ❌                       |
| Lazy         | ✅    | ❌           | ✅           | ❌                    | ❌                       |
| Synchronized | ✅    | ✅           | ❌           | ❌                    | ❌                       |
| DCL          | ✅    | ✅           | ✅           | ❌                    | ❌                       |
| Bill Pugh    | ✅    | ✅           | ✅           | ❌                    | ❌                       |
| Enum         | ✅    | ✅           | ✅           | ✅                    | ✅                       |

---

# 4. Interview Deep Insights

---

##  Why not always use Singleton?
- Hidden dependencies (global state)
- Hard to test (mocking issues)
- Violates **Single Responsibility Principle**

---
##  When to use?
- Shared resource management
- Stateless services
- Configuration objects

---

# 5. Tricky Interview Questions (with Answers)

---

## 1. Why is Double Checked Locking broken before Java 5?
- Due to **memory model issues**
- Instruction reordering → partially initialized object
- Fixed using `volatile` in Java 5+
## 2. Can Singleton be broken?
Yes:
- Reflection
- Serialization
- Cloning
Only **Enum Singleton is fully safe**
##  3. Why is Bill Pugh better than DCL?
- No synchronization overhead
- Simpler
- JVM guarantees safety via class loading
## 4. Can we make Singleton lazy AND thread-safe without sync?
Yes → **Bill Pugh method**
## 5. Why Enum Singleton is preferred in modern Java?
- Serialization-safe by default
- Reflection-proof
- Simpler and cleaner
## 6. What happens if Singleton implements Serializable but no readResolve()?
- Deserialization creates **new object** 
- Breaks Singleton guarantee
## 7. Is Singleton per JVM or per ClassLoader?
- **Per ClassLoader** 
- Multiple classloaders → multiple instances
## 8. How does Spring handle Singleton?
- Default scope = Singleton
- But managed by container → NOT strict GoF singleton
## 9. Can we lazy load Enum Singleton?
- No explicit lazy loading
- But class loading is lazy → effectively lazy
## 10. How to prevent reflection breaking Singleton?
```java
private Singleton() {
    if (instance != null) {
        throw new RuntimeException("Use getInstance()");
    }
}
```

---

# 6.  Advanced Edge Cases

---
##  Multiple ClassLoaders
- App servers → multiple instances possible
- Each classloader = separate namespace
## Distributed Systems
- Singleton is NOT global across services
- Need:
    - Redis locks        
    - Zookeeper
    - DB locks
##  Multithreading Visibility
- Without `volatile`:
    - Thread sees stale value
