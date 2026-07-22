#  1. Reflection in Java

---
##  1. What is Reflection?
Reflection allows a Java program to:
- Inspect classes, methods, fields **at runtime**
- Modify behavior dynamically
Works via:
- `java.lang.Class`
- `java.lang.reflect.*`
##  2. Core Use Cases
- Frameworks (Spring, Hibernate)
- Dependency Injection
- Serialization/Deserialization
- Testing (JUnit)
##  3. Basic Example
```java
Class<?> clazz = Class.forName("com.example.MyClass");

Method method = clazz.getDeclaredMethod("sayHello");
method.setAccessible(true);

Object obj = clazz.getDeclaredConstructor().newInstance();
method.invoke(obj);
```
##  4. Key Reflection APIs

| API           | Purpose           |
| ------------- | ----------------- |
| `Class`       | Metadata of class |
| `Field`       | Access fields     |
| `Method`      | Invoke methods    |
| `Constructor` | Create instances  |
## 5.  Internal Working
- JVM maintains **metadata in Method Area**
- Reflection reads/modifies metadata
- Uses **native calls + security checks**
## 6. Problems with Reflection

### 1. Performance overhead
- Slower than direct calls (no inlining, JIT optimizations limited)
### 2.Breaks encapsulation
```java
field.setAccessible(true);
```
### 3. Security risks
- Can access private members
## 7. Reflection Breaking Singleton

```java
Constructor<Singleton> c = Singleton.class.getDeclaredConstructor();
c.setAccessible(true);
Singleton s2 = c.newInstance(); // breaks singleton
```

---
##  Interview Deep Insights

---
### 1. Why is reflection slow?
- No compile-time optimizations
- Uses dynamic resolution
- JVM cannot inline reflective calls

### 2. Can JIT optimize reflection?
- Limited optimization
- Some caching improvements in newer JVMs
- Still slower than direct calls
### 3. Where is reflection heavily used?
- Spring (DI, AOP)
- Hibernate (ORM mapping)
- Jackson (JSON parsing)
### 4. What is `setAccessible(true)`?
- Disables access checks
- Allows private access
- Can break encapsulation

#  2. Heap Pollution (Very Important for Interviews)

---
##  1. What is Heap Pollution?
👉 Occurs when:
> A variable of a **parameterized type** refers to an object that is not of that type
##  2. Root Cause
- Type erasure + raw types
- Generics not enforced at runtime
## 3. Example

```java
List<String> list = new ArrayList<>();
List rawList = list; // raw type

rawList.add(100); // Heap pollution

String s = list.get(0); // ClassCastException
```

## 4. Why it happens?
- Generics are erased at runtime
- JVM sees only `List`, not `List<String>`
## 5. Another Dangerous Case (Varargs)

```java
@SafeVarargs
static void unsafe(List<String>... lists) {
    Object[] array = lists;
    array[0] = List.of(1, 2, 3); // heap pollution
}
```
## 6. Key Concept: Type Erasure
- Compiler removes generic info
- Replaces with `Object` or bounds
## 7. Prevention
- Use generics properly
- Avoid raw types
- Use `@SafeVarargs`

---

##  8. Interview Deep Insights

### 1. Why does Java allow heap pollution?
- Backward compatibility with pre-generics code
### 2. Why does it fail at runtime, not compile time?
- Due to **type erasure**
- Compiler cannot enforce runtime type safety
### 3. What is `@SafeVarargs`?
- Suppresses unchecked warnings
- Ensures method does not perform unsafe operations
---

#  3. Annotations in Java

---
## 1. What are Annotations?
Metadata added to:
- Classes
- Methods
- Fields

Used by:
- Compiler
- Runtime frameworks

## 2. Built-in Annotations

| Annotation          | Purpose                    |
| ------------------- | -------------------------- |
| `@Override`         | Method override check      |
| `@Deprecated`       | Marks obsolete             |
| `@SuppressWarnings` | Suppress compiler warnings |
## 3. How Annotations Work Internally
- Stored in `.class` file
- JVM reads via reflection
- Example
```java
@Override
public String toString() {
    return "Hello";
}
```

---
#  4. Custom Annotations

---
## 1. Creating Custom Annotation
```java
@interface MyAnnotation {
    String value();
}
```

## 2. Must Define Meta-Annotations

- Example (Full)
```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@interface LogExecutionTime {
}
```
## 3. Usage

```java
@LogExecutionTime
public void process() {}
```

## 4. Reading via Reflection

```java
Method method = obj.getClass().getMethod("process");

if (method.isAnnotationPresent(LogExecutionTime.class)) {
    System.out.println("Logging...");
}
```


---

#  5. Meta-Annotations (VERY IMPORTANT)

---

## 1. What are Meta-Annotations?
Annotations on annotations

### 1️. `@Retention`
Defines lifecycle

| Type    | Meaning                   |
| ------- | ------------------------- |
| SOURCE  | Discarded at compile time |
| CLASS   | Stored in class file      |
| RUNTIME | Available at runtime      |

### 2️. `@Target`

Where annotation can be used
```java
@Target(ElementType.METHOD)
```
### 3️. `@Documented`
- Included in Javadoc
### 4️. `@Inherited`
- Child class inherits annotation    
### 5️. `@Repeatable`
Allows multiple annotations
```java
@Repeatable(Authors.class)
@interface Author {
    String name();
}
```

---

## 2. Interview Deep Insights

---
### 1. Why is `RetentionPolicy.RUNTIME` important?
- Required for reflection 
- Without it, frameworks cannot read annotation
### 2. Difference between SOURCE, CLASS, RUNTIME?
- SOURCE → only compiler
- CLASS → bytecode
- RUNTIME → reflection available
### 3. Can annotations change program behavior?
- Not directly
- But frameworks interpret them and act
### 4. How does Spring use annotations?\
- Uses reflection + proxies
- Example:
    - `@Autowired`
    - `@Transactional`

---

# 3. Tricky Interview Questions (High Signal)

---
## 1. Can annotations exist without reflection?
Yes:
- Compiler-level annotations (`@Override`)
- Reflection needed only for runtime use
## 2. Why annotations cannot have method bodies?
- They are metadata, not behavior
## 3. Can we extend annotations?
- No (they implicitly extend `java.lang.annotation.Annotation`)
## 4. What happens if two annotations conflict?
- Framework-specific behavior
## 5. Why generics + varargs cause heap pollution?
- Arrays are covariant
- Generics are invariant
- Leads to unsafe assignments
## 6. Is reflection compile-time or runtime?
- Purely runtime
## 7. Can reflection modify final fields?
- Yes (with hacks), but unsafe
## 8. Why annotations are preferred over XML?
- Type-safe
- Cleaner
- Compile-time validation
