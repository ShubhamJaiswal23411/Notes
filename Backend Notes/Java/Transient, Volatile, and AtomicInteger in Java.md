## 1. `transient` Keyword

### Definition
- The `transient` keyword in Java is used to **indicate that a field should not be serialized**.
- When an object is serialized, transient fields are ignored and **not persisted**.
- Commonly used for **sensitive information** (like passwords) or **non-serializable fields**.
### Key Points
- Only affects **serialization** (`ObjectOutputStream` / `Serializable`).
- Does **not affect runtime behavior**.
- Default value is assigned during deserialization:
    - `null` for objects
    - `0` for numeric types
    - `false` for boolean  

### Example

```java
import java.io.*;

class User implements Serializable {
    private String name;
    private transient String password; // won't be serialized

    public User(String name, String password) {
        this.name = name;
        this.password = password;
    }

    public String toString() {
        return "User{name='" + name + "', password='" + password + "'}";
    }
}

public class TransientExample {
    public static void main(String[] args) throws Exception {
        User user = new User("Alice", "secret123");

        // Serialize
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("user.ser"));
        oos.writeObject(user);
        oos.close();

        // Deserialize
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream("user.ser"));
        User deserializedUser = (User) ois.readObject();
        ois.close();

        System.out.println(deserializedUser);
        // Output: User{name='Alice', password='null'}
    }
}
```

### Tricky Interview Questions

1. **Can a `transient` field be serialized manually?**
    - Yes, by implementing `writeObject()` and `readObject()` methods for custom serialization.
    
2. **What happens if a `transient` field is `final`?**
    - It cannot be serialized normally, but can still be initialized in `readObject()`.


---

## 2. `volatile` Keyword

### Definition
- The `volatile` keyword in Java is used to indicate that a **variable may be modified by multiple threads**.
- Ensures **visibility** of changes across threads.
### Key Points
- Guarantees **visibility** but **not atomicity**.
- Avoids caching of the variable in CPU registers or thread-local memory.
- Often used in **double-checked locking** and **flags for stopping threads**.

### Example
```java
class VolatileExample {
    private volatile boolean running = true;

    public void run() {
        new Thread(() -> {
            while (running) {
                // do some work
            }
            System.out.println("Thread stopped.");
        }).start();
    }

    public void stop() {
        running = false; // visibility guaranteed
    }

    public static void main(String[] args) throws InterruptedException {
        VolatileExample example = new VolatileExample();
        example.run();
        Thread.sleep(1000);
        example.stop();
    }
}
```

### Limitations
- **Atomic operations** like `count++` are **not guaranteed** to be safe even if `count` is volatile.
- Use `AtomicInteger` or `synchronized` for atomicity.
### Tricky Interview Questions
1. **Is `volatile` enough for compound actions like increment (`i++`)?**
    - No. `volatile` ensures visibility but not atomicity. Use `AtomicInteger`.

2. **Can `volatile` be used with objects?**
    - Yes, but only the reference is volatile; the object’s internal state is not automatically thread-safe.

3. **Why does `double-checked locking` need `volatile`?**
    - Without it, the JVM may reorder instructions, leading to partially constructed objects being visible to other threads.

---

## 3. `AtomicInteger` Class

### Definition
- Part of `java.util.concurrent.atomic` package.
- Provides **thread-safe, lock-free operations** on integers.
- Implements **CAS (Compare-And-Swap)** internally for atomic updates.
### Key Points
- Supports atomic operations: `incrementAndGet()`, `getAndIncrement()`, `addAndGet()`, `compareAndSet()`.
- **No synchronization needed** for single variable updates.
- Useful in **counters, flags, and accumulators** in concurrent programming.

### Example
```java
import java.util.concurrent.atomic.AtomicInteger;

class AtomicIntegerExample {
    private AtomicInteger counter = new AtomicInteger(0);

    public void increment() {
        counter.incrementAndGet(); // atomic
    }

    public int getCounter() {
        return counter.get();
    }

    public static void main(String[] args) throws InterruptedException {
        AtomicIntegerExample example = new AtomicIntegerExample();

        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) example.increment();
        });

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) example.increment();
        });

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("Counter: " + example.getCounter()); 
        // Output: Counter: 2000
    }
}
```

### Tricky Interview Questions

1. **Difference between `AtomicInteger` and `volatile int`?**
    - `volatile` ensures visibility but not atomicity. `AtomicInteger` ensures both atomicity and visibility.

2. **Can `AtomicInteger` replace `synchronized` entirely?**    
    - Only for simple atomic operations on a single variable. Complex compound operations may still need locks.

3. **Explain how CAS works in `AtomicInteger`.**
    - Reads the value, compares with expected, updates if equal. Repeats if another thread has modified it.


---

## 4. Comparison Table

| Feature          | transient                         | volatile                         | AtomicInteger                         |
| ---------------- | --------------------------------- | -------------------------------- | ------------------------------------- |
| Purpose          | Exclude field from serialization  | Ensure visibility across threads | Atomic, thread-safe operations on int |
| Thread Safety    | N/A                               | Only visibility, not atomic      | Atomic operations                     |
| Atomicity        | N/A                               | No                               | Yes                                   |
| Typical Use Case | Sensitive info, non-serial fields | Stop flag, shared boolean        | Counters, accumulators                |
| JVM Handling     | Ignored during serialization      | Avoids caching & reordering      | CAS (Compare-And-Swap) internally     |



---

## 1. Core Difference between AtomicInteger and volatile

| Feature               | volatile                          | AtomicInteger                               |
| --------------------- | --------------------------------- | ------------------------------------------- |
| **Purpose**           | Visibility across threads         | Visibility **and atomicity**                |
| **Thread Safety**     | Only ensures latest value is read | Ensures atomic read-modify-write operations |
| **Atomic Operations** | ❌ Not atomic (`i++` not safe)     | ✅ Atomic (`incrementAndGet()`)              |
| **Use Case**          | Flags, stop signals               | Counters, accumulators                      |

**Key Idea:**
- `volatile` = “all threads see the latest value, but updates are **not safe if multiple threads change it simultaneously**.”
- `AtomicInteger` = “all threads see the latest value, and you can safely update it **without locks**.”

---
## 2. Example to Show Difference

### Using `volatile`

```java
class VolatileCounter {
    private volatile int count = 0;

    public void increment() {
        count++; // not atomic
    }

    public int getCount() {
        return count;
    }

    public static void main(String[] args) throws InterruptedException {
        VolatileCounter counter = new VolatileCounter();

        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) counter.increment();
        });
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) counter.increment();
        });

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("Volatile count: " + counter.getCount());
    }
}
```

**What happens:**
- You might **expect 2000**, but the output is often **less than 2000**.
- Reason: `count++` is **not atomic**; it’s actually **three steps**:
    1. Read the value
    2. Add 1
    3. Write back
- If two threads read the same value at the same time, one update is lost.

---
### Using `AtomicInteger`
```java
import java.util.concurrent.atomic.AtomicInteger;

class AtomicCounter {
    private AtomicInteger count = new AtomicInteger(0);

    public void increment() {
        count.incrementAndGet(); // atomic
    }

    public int getCount() {
        return count.get();
    }

    public static void main(String[] args) throws InterruptedException {
        AtomicCounter counter = new AtomicCounter();

        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) counter.increment();
        });
        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 1000; i++) counter.increment();
        });

        t1.start();
        t2.start();
        t1.join();
        t2.join();

        System.out.println("Atomic count: " + counter.getCount());
    }
}
```

**What happens:**
- The output **will always be 2000**.
- Reason: `incrementAndGet()` uses **Compare-And-Swap (CAS)** internally → it reads the value, computes the new value, and updates **only if no other thread changed it in the meantime**. No updates are lost.

---

## 3. Practical Use Cases

- **volatile**
    - Stopping a thread safely
    - Flag variables like `volatile boolean running = true`

- **AtomicInteger**   
    - Thread-safe counters in multithreading
    - Implementing lock-free algorithms

