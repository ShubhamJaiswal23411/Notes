
# 1️. Lambda Expressions

##  1. What It Is

A way to write **anonymous functions** (functions without a name).

```java
(a, b) -> a + b
```

Used mainly with **functional interfaces**.

---
## 2. Why It Was Added

Before Java 8:

```java
Collections.sort(list, new Comparator<Integer>() {
    public int compare(Integer a, Integer b) {
        return a - b;
    }
});
```

Too verbose.
Java 8 introduced lambdas to:
- Reduce boilerplate
- Enable functional-style programming
- Improve readability
- Prepare for Stream API

---

## 3. Impact
- Less code
- More expressive APIs
- Enabled parallel processing patterns

---

# 2️. Functional Interfaces

## 1. What It Is

An interface with **exactly one abstract method**.
Example:
```java
@FunctionalInterface
interface MyFunc {
    void apply();
}
```

---

## 2. Built-in Functional Interfaces (java.util.function)

| Interface           | Purpose               |
| ------------------- | --------------------- |
| `Predicate<T>`      | Boolean condition     |
| `Function<T,R>`     | Takes T, returns R    |
| `Consumer<T>`       | Takes T, returns void |
| `Supplier<T>`       | Returns T             |
| `UnaryOperator<T>`  | T → T                 |
| `BinaryOperator<T>` | (T,T) → T             |

---

## 3. Why Added

To standardize lambda usage and enable reusable functional patterns.

---

# 3️. Stream API

## 1. What It Is

A **declarative pipeline API** for processing collections.
```java
list.stream()
    .filter(x -> x > 10)
    .map(x -> x * 2)
    .collect(Collectors.toList());
```

---

## 2. Why It Was Added

Before Java 8:
- Imperative loops
- Manual iteration
- Hard to parallelize

Stream API provides:
- Declarative style
- Lazy evaluation
- Internal iteration
- Easy parallel processing

---

## 3. Core Concepts

### Intermediate Operations
- `filter()`
- `map()`
- `sorted()`
- `distinct()`
### Terminal Operations
- `collect()`
- `forEach()`
- `reduce()`
- `count()`

---

## 4. Parallel Streams

```java
list.parallelStream()
```

Uses ForkJoinPool internally.

---

## 5. Impact
- Cleaner data processing
- Functional programming model
- Simplified concurrency

---

# 4️. Default Methods in Interfaces

## 1. What It Is

Interfaces can now have method implementations.
```java
interface A {
    default void show() {
        System.out.println("Hello");
    }
}
```

---

## 2. Why It Was Added

To support **backward compatibility**.
Problem:  
When adding new methods to interfaces like `List`, all implementing classes would break.

Solution:  
Default methods allow adding methods without breaking existing code.

---

## 3. Impact

- Evolved collections framework
- Enabled Stream API integration

---

# 5️. Static Methods in Interfaces

Interfaces can now have static methods:
```java
interface A {
    static void greet() { }
}
```

---

## 1. Why Added

To group utility methods with interface definitions. They provide implementations of thing that dont change, since they are static they cant be overridden either.

---

# 6️. Method References

## 1. What It Is

Short-hand lambda for existing methods.
```java
System.out::println
```

Types:

| Type        | Example               |
| ----------- | --------------------- |
| Static      | `Class::staticMethod` |
| Instance    | `obj::method`         |
| Constructor | `Class::new`          |

---

## 2. Why Added

- Improve readability
- Reduce lambda verbosity

---

# 7️. Optional Class

## 1. What It Is

A container to handle **null safely**.
```java
Optional<String> name = Optional.of("John");
```

---

## 2. Why It Was Added

To reduce:
- NullPointerException
- Defensive null checks

Encourages:
- Explicit absence handling
- Functional-style chaining

---

## 3. Key Methods

- `isPresent()`
- `orElse()`
- `orElseGet()`
- `orElseThrow()`
- `map()`

---

# 8️. New Date & Time API (java.time)

## 1. What It Is

Modern, immutable date-time library.
Inspired by Joda-Time.
Main classes:
- `LocalDate`
- `LocalTime`
- `LocalDateTime`
- `ZonedDateTime`
- `Instant`
- `Period`
- `Duration`

---

## 2. Why Added

Old API problems:
- `Date` was mutable
- `Calendar` was complex
- Not thread-safe
- Poor timezone handling

New API provides:
- Immutable objects
- Clear API design
- Better timezone support
- ISO-8601 standard compliance

---

## 3. Impact

Huge improvement in date handling.


---

# 9. Base64 API

New utility class:

```java
Base64.getEncoder().encode()
```

---

## 1. Why Added

Previously required third-party libraries.
Now built into Java.

# 1️0. CompletableFuture

Asynchronous programming support.

```java
CompletableFuture.supplyAsync(() -> "Hello")
                 .thenApply(s -> s + " World");
```

---

## 🔹 Why Added
- Improve concurrency model
- Non-blocking programming
- Replace complex Future handling

---

# 1️1. Improvements to Collections API

Added methods like:
- `forEach()`
- `removeIf()`
- `replaceAll()`
- `computeIfAbsent()`
- `merge()`

