
## **1. @NoArgsConstructor**
- Generates a **no-argument constructor** (default constructor).
- Useful for frameworks like **Spring, JPA/Hibernate**, which often require a no-arg constructor.
- Can be combined with `@Entity` for JPA entities.
**Example:**
```java
@NoArgsConstructor
public class Car {
    private String name;
    private int year;
}
```

Generates:
```java
public Car() {
}
```

**Notes:**
- Required for **JPA entities** because Hibernate instantiates objects via reflection.
- Fields are **not initialized** unless explicitly assigned defaults.
- Can also be used with `force=true` to initialize `final` fields with default values.
```java
@NoArgsConstructor(force = true)
public class Car {
    private final String name;
}
```

---
## **2. @AllArgsConstructor**
- Generates a constructor with **all fields as parameters**.
- Useful when you want to **create fully-initialized objects** quickly.

**Example:**
```java
@AllArgsConstructor
public class Car {
    private String name;
    private int year;
}
```

Generates:
```java
public Car(String name, int year) {
    this.name = name;
    this.year = year;
}
```

**Notes:**
- Works well for **DTOs, value objects**, or classes where all fields are mandatory.
- For **JPA entities**, be careful: using this alone can bypass **default constructor requirements**.

---
## **3. @RequiredArgsConstructor**
- Generates a constructor for **all `final` fields or fields marked `@NonNull`**.
- Useful for **dependency injection and immutability**.

**Example:**
```java
@RequiredArgsConstructor
public class CarService {
    private final Engine engine;
    private String name; // not final → not included
}
```

Generates:
```java
public CarService(Engine engine) {
    this.engine = engine;
}
```

**Notes:**
- Ideal for **constructor injection** in Spring beans.
- Helps create **immutable classes** if all required fields are `final`.
- Fields not `final` or `@NonNull` → ignored.

---

## **4. Comparison Table**

| Annotation                 | Constructor Generated                        | Typical Use Case                                       |
| -------------------------- | -------------------------------------------- | ------------------------------------------------------ |
| `@NoArgsConstructor`       | No-arg constructor                           | JPA entities, frameworks that need default constructor |
| `@AllArgsConstructor`      | Constructor with **all fields**              | DTOs, value objects, quick initialization              |
| `@RequiredArgsConstructor` | Constructor with **final & @NonNull fields** | Dependency injection, immutable classes                |
