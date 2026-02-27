
# All 11 Methods of `java.lang.Object`

Every class in Java implicitly extends `Object`.

Here are all the methods defined in `Object`:

---

## 1. `equals(Object obj)`
- Compares logical equality.
- Default → compares memory addresses (same as `==`).

---
## 2️. `hashCode()`
- Returns integer hash value.
- Must be consistent with `equals()`.

---
## 3️. `toString()`
- Returns string representation.
- Default → `ClassName@hexHashCode`.

---
## 4️. `getClass()`
- Returns runtime class (`Class<?>`).
- Final method.

---
## 5️. `clone()`
- Creates copy of object.
- Protected.
- Requires `Cloneable` interface.

---
## 6️. `finalize()` (Deprecated since Java 9)
- Called by GC before object destruction.
- Should NOT be used in modern Java.

---
## 7️. `wait()`
- Causes current thread to wait.
- Must be called inside synchronized block.

---
## 8️. `wait(long timeout)`
- Wait with timeout.
    

---
## 9️. `wait(long timeout, int nanos)`
- Wait with timeout + nanoseconds.

---

## 10. `notify()`
- Wakes one waiting thread.

---
## 11. `notifyAll()`
- Wakes all waiting threads.

---

# 🔥 Common Interview Tricky Questions on `Object` Class

---

## 1. Why must we override both `equals()` and `hashCode()`?
Because:
> If two objects are equal → they MUST have same hashCode.

If not:
- HashMap / HashSet breaks
- Objects may not be found in collections

---
##  2. Can we override `getClass()`?
 No — it is `final`.

---

## 3. Can we override `wait()` or `notify()`?
Technically yes (not final), but practically **never done**.
They are native methods and tied to monitor mechanism.

---

##  4. Difference between `==` and `equals()`?

| `==`                 | `equals()`         |
| -------------------- | ------------------ |
| Reference comparison | Logical comparison |
| Cannot override      | Can override       |

---

##  5. Why is `clone()` protected?
To prevent cloning unless explicitly allowed.
You must:
- Implement `Cloneable`
- Override `clone()` and make it public

---
##  6. What happens if you override `equals()` but not `hashCode()`?
Example:
```java
Map<Person, String> map = new HashMap<>();
```
You won’t be able to retrieve object properly.
Breaks hash-based collections.

---
##  7. Why is `wait()` in Object class and not Thread?
Because:
> Waiting happens on object monitors (locks), not threads.

Every object has a monitor.

---
##  8. Can `hashCode()` return negative number?
✅ Yes.
Hash codes can be negative.

---
##  9. Can two unequal objects have same hashCode?
✅ Yes (hash collision).
But equal objects must have same hashCode.

---
##  10. Is `toString()` automatically called?
Yes, when:
- Printing object
- Logging
- String concatenation
---
##  11. What is default implementation of `equals()`?
In Object:
```java
public boolean equals(Object obj) {
    return (this == obj);
}
```

---
##  12. Is `clone()` deep copy?
❌ No.
Default clone is shallow copy.

---

##  13. Can we call `wait()` without synchronized?
No.
Throws:

```
IllegalMonitorStateException
```

---
##  14. Why is `notifyAll()` safer than `notify()`?
Because:
- `notify()` wakes only one thread (may cause deadlock)
- `notifyAll()` wakes all waiting threads

---
##  15. Is `Object` class abstract?
No.
It is a concrete class.

