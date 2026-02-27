
# 1️. What is Immutability?

An object is **immutable** if its state cannot change after it is created.
Once constructed:
- No field can be modified
- No internal data can be altered

Examples:
- `String`
- `Integer`
- `LocalDate`

---

# 2. How to Create an Immutable Class

## 🔹 Rules to Follow
1. Make the class `final`
2. Make all fields `private` and `final`
3. No setters
4. Initialize fields through constructor
5. Perform defensive copy for mutable fields
6. Return defensive copies in getters

---

##  Correct Implementation

```java
import java.util.Date;

public final class Person {

    private final String name;
    private final Date birthDate; // mutable field

    public Person(String name, Date birthDate) {
        this.name = name;
        this.birthDate = new Date(birthDate.getTime()); // defensive copy
    }

    public String getName() {
        return name;
    }

    public Date getBirthDate() {
        return new Date(birthDate.getTime()); // defensive copy
    }
}
```

---

#  2️. Advantages & Disadvantages of Immutability

---

## Advantages

### 1️. Thread Safety
No synchronization required.

### 2️. Safe as HashMap Keys
Hash code never changes.

### 3️. Easy to Cache
Can reuse objects safely.

### 4️. Predictable Behavior
No unexpected state changes.

### 5️. Better Security
Prevents accidental modification.

---

##  Disadvantages

### 1️. Memory Overhead
Every modification creates new object.

### 2️. Performance Cost
Extra object creation.

### 3️. Not Suitable for Large Mutable Data
e.g., large collections frequently modified.

---

# 3️. Which Classes Are Immutable in Java?

---

##  Wrapper Classes
- `Integer`
- `Long`
- `Double`
- `Boolean`
- `Byte`
- `Short`
- `Character`

---

## String
- `String`

---

## Java Time API
- `LocalDate`
- `LocalDateTime`
- `Instant`
- `ZonedDateTime`

---

##  Big Number Classes
- `BigInteger`
- `BigDecimal`

---

#  4️. Are `new String()` Objects Also Immutable?

Yes.
```java
String s1 = new String("hello");
```

Even though:
- It creates new object in heap
- Not reused from string pool

It is still immutable because:
- Internal `char[]` (or `byte[]` in newer Java) cannot be modified externally
- No setter methods

Difference is only memory behavior, not immutability.

---

#  5️. Common Tricky Interview Questions on Immutability

---

## 1. Is String truly immutable?

Yes.
Even though it internally uses array:
- Array is private
- No direct access
- No modification methods

---

##  2. Why is String immutable?

1. Security (class loading, URLs, file paths)
2. Thread safety
3. String pool optimization
4. HashCode caching
5. Safe for HashMap keys

---

##  3. Can we break immutability using reflection?

Technically yes (advanced hack), but not in normal use.

> Immutability holds under normal usage, not under reflection attacks.

---

##  4. Why is immutability important for HashMap keys?

If key changes:
- hashCode changes
- Bucket changes
- Retrieval fails

---

## 5. Are records immutable?
Java records:
- Fields are final
- No setters  
    But:  
    If they contain mutable objects → immutability not guaranteed.

---

## 6. Is `Collections.unmodifiableList()` immutable?
No.
It is unmodifiable view, but:
- Original list can still change.

---

## 7. Why are wrapper classes immutable?
To:
- Enable caching (Integer cache)
- Safe usage in collections
- Thread safety

