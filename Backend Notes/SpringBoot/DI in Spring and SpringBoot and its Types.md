
# **Dependency Injection (DI) in Spring**

**Dependency Injection** is a design pattern used in Spring to achieve **Inversion of Control (IoC)**. Instead of creating dependencies inside a class, Spring injects the dependencies externally. This promotes **loose coupling**, **testability**, and **maintainability**.

---
## **1. Types of Dependency Injection in Spring**
Spring supports multiple ways to inject dependencies into a class:
1. **Constructor Injection**
2. **Setter Injection**
3. **Field Injection (`@Autowired`)**
4. **Interface Injection (rarely used in Spring)**

---
## **2. @Autowired Injection**
- `@Autowired` is a Spring annotation used to **automatically wire a bean**.
- Can be applied to:
    - Fields
    - Setter methods
    - Constructors

**Example: Field Injection**
```java
@Component
public class CarService {

    @Autowired
    private Engine engine;

    public void startCar() {
        engine.start();
    }
}
```

**Notes:**
- Spring automatically injects the dependency (`Engine`) into the field.
- No need for constructors or setters explicitly.
- Requires **Spring-managed beans** (`@Component`, `@Service`, `@Repository`, `@Bean`).

---
## **3. Constructor Injection**
- Dependencies are provided through the class constructor.
- Recommended for **mandatory dependencies**
- Supports **immutability** (final fields).
- Works well with **unit testing**.

**Example:**
```java
@Component
public class CarService {

    private final Engine engine;

    @Autowired
    public CarService(Engine engine) { // Constructor Injection
        this.engine = engine;
    }

    public void startCar() {
        engine.start();
    }
}
```

**Notes:**
- If there’s only **one constructor**, `@Autowired` is optional in Spring 4.3+.
- Encourages **final fields**, making classes **immutable**.
- **Best practice** in modern Spring applications.

---
## **4. Setter Injection**
- Dependencies are provided via **setter methods**.
- Useful for **optional dependencies**.
- Supports **partial injection**.

**Example:**
```java
@Component
public class CarService {

    private Engine engine;

    @Autowired
    public void setEngine(Engine engine) { // Setter Injection
        this.engine = engine;
    }

    public void startCar() {
        engine.start();
    }
}
```

**Notes:**
- Good for optional or changeable dependencies.
- Not ideal for mandatory dependencies.
- Allows **re-injection** of dependencies.

---
## **5. Comparison: @Autowired vs Constructor vs Setter Injection**

| Feature               | @Autowired (Field)                              | Constructor Injection                 | Setter Injection                     |
| --------------------- | ----------------------------------------------- | ------------------------------------- | ------------------------------------ |
| **Definition**        | Auto-inject dependency directly into field      | Inject dependency through constructor | Inject dependency via setter method  |
| **Required/Optional** | Mandatory by default (can use `required=false`) | Mandatory by default                  | Can be optional                      |
| **Immutability**      | Not supported                                   | Supported (use `final`)               | Not supported                        |
| **Testability**       | Harder to test (reflection needed)              | Easier (pass mock via constructor)    | Moderate (set mock via setter)       |
| **Code Verbosity**    | Less verbose                                    | Slightly more verbose                 | More verbose                         |
| **When to use**       | Quick prototyping, small apps                   | Recommended standard for mandatory DI | Optional dependencies or legacy code |

---

## **6. Other Types of Dependency Injection in Spring**
1. **Interface Injection**
    - Rarely used in Spring.
    - Class implements an interface to accept dependency.
2. **`@Resource` Injection**
    - Part of **JSR-250**.
    - Injects dependency by **name** instead of type.
    ```java
    @Resource(name="engineBean")
    private Engine engine;
    ```
3. **`@Inject` Injection**
    - Part of **JSR-330**, similar to `@Autowired`.
    ```java
    @Inject
    private Engine engine;
    ```
4. **XML-based Injection**
    - Legacy approach using **XML configuration**.
    ```xml
    <bean id="carService" class="com.example.CarService">
        <property name="engine" ref="engineBean"/>
    </bean>
    ```
5. **Java-based Configuration Injection**
    - Using `@Bean` methods in `@Configuration` classes.
    ```java
    @Configuration
    public class AppConfig {
        @Bean
        public Engine engine() {
            return new Engine();
        }
    
        @Bean
        public CarService carService() {
            return new CarService(engine());
        }
    }
    ```

---

## **7. Best Practices in Spring Dependency Injection**

1. Prefer **constructor injection** for mandatory dependencies.
2. Use **setter injection** for optional dependencies.
3. Avoid **field injection** for production code (makes testing harder).
4. Make dependencies **final** whenever possible for immutability.
5. Use **`@Autowired(required=false)`** if a dependency is optional.
6. Leverage **Java config** over XML in modern Spring Boot applications.

---
## **8. Quick Summary Table**

|Injection Type|Mandatory/Optional|Best Use Case|Testable|Supports Immutability|
|---|---|---|---|---|
|Field (`@Autowired`)|Mandatory|Quick prototyping|Low|No|
|Constructor|Mandatory|Recommended standard|High|Yes|
|Setter|Optional|Optional or changeable dependency|Medium|No|
|`@Resource`|Name-based|When bean name is important|Medium|No|
|`@Inject`|Type-based|Java EE style|High|Yes|
|XML|Type or name|Legacy projects|Medium|No|

---

 **Key Takeaways:**
- Constructor injection = standard & recommended.
- Setter injection = optional dependencies.
- Field injection (`@Autowired`) = convenient, but less testable.
- Spring supports multiple ways: annotations, XML, and Java config.

---

## **9. Testability of @Autowired vs Constructor vs Setter Injection**

The key idea here is: **how easy is it to provide mock dependencies for unit testing?**
### **1. Field Injection (@Autowired)**

```java
@Component
public class CarService {
    @Autowired
    private Engine engine;

    public void startCar() {
        engine.start();
    }
}
```

- Problem: `engine` is **private** and **Spring injects it via reflection**, so your test can’t simply `new CarService()` and pass a mock `Engine`.
- To test, you usually need:
    - Spring context (`@SpringBootTest`)  
        **OR**
    - Reflection utilities (like `ReflectionTestUtils.setField(...)`) to inject the mock.
- **Why harder:** You can’t create the class without Spring; test becomes slower or more complex.

---

### **2. Constructor Injection**

```java
public class CarService {
    private final Engine engine;

    public CarService(Engine engine) { 
        this.engine = engine;
    }

    public void startCar() {
        engine.start();
    }
}
```

- Testable: You can easily do:
```java
Engine mockEngine = Mockito.mock(Engine.class);
CarService carService = new CarService(mockEngine);
```

- No Spring needed.
- **Why easier:** The dependency is explicit in the constructor.

---

### **3. Setter Injection**

```java
public class CarService {
    private Engine engine;

    public void setEngine(Engine engine) { 
        this.engine = engine;
    }

    public void startCar() {
        engine.start();
    }
}
```

- Testable: You can create the object and then call the setter:
```java
CarService carService = new CarService();
carService.setEngine(mockEngine);
```

- **Moderate:** Still manual step; not as clean as constructor injection.

 **Summary of Testability:**

| Injection Type       | How to test without Spring context                 |
| -------------------- | -------------------------------------------------- |
| Field (`@Autowired`) | Hard: need reflection or Spring context            |
| Constructor          | Easy: pass mocks in constructor                    |
| Setter               | Medium: call setter manually after object creation |
## Immutability with Constructor Injection

- As far as the immutability goes we are not achieving full immutability when we make fields final and use a constructor injection (either write the constructors ourself or use @RequiredArgsConstructor)
- Things that are breaking immutability , 
	- class is not final so it can be extended 
	- fields are final but they still can be mutable like a list , in this case we can use unmodifiable list
	- we are not making a deep copy when setting the field so the fields can still be changed with original reference but the good thing is we don't have the original reference since ioc container is the only entity that has them so this point is safe.
	- we can still add setters if we want to which breaks immutability.
