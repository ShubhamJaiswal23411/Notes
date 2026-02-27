

## 1. Where it can be used ?
### **1. On a Bean Class**

When you annotate a bean class with `@Lazy`:

```java
@Lazy
@Component
public class MyBean { ... }
```

- Spring **does not create an instance of this bean at startup**.
- The bean is instantiated **only when it is first requested** (e.g., injected into another bean or retrieved from the `ApplicationContext`).
- This can improve **startup time** if the bean is heavy or not always needed.

---
### **2. On a Constructor**

If you annotate a constructor with `@Lazy`:
```java
@Component
public class MyBean {
    @Lazy
    @Autowired
    public MyBean(AnotherBean anotherBean) { ... }
}
```

- The effect is essentially the same as annotating the bean itself.
- Spring **delays the instantiation of this bean** until it is first required.
- Dependencies are initialized only when the bean is created.

---

### **3. On a Dependency**

You can also annotate a **dependency** in another bean:
```java
@Component
public class MyService {
    private final MyBean myBean;

    @Autowired
    public MyService(@Lazy MyBean myBean) {
        this.myBean = myBean;
    }
}
```

- Here, `MyBean` will **not be created when `MyService` is instantiated**.
- Instead, Spring creates a **proxy** for `MyBean`.
- The actual `MyBean` instance is created **only when a method on the proxy is first called**.

---

### **Key Points**

- `@Lazy` **delays bean instantiation** until it is actually needed.
- Useful for **reducing startup time** or **avoiding circular dependencies**.
- Works with **class-level beans**, **constructors**, or **injected dependencies**.
- If multiple beans depend on a `@Lazy` bean, they all get the same lazy-initialized instance (singleton behavior is preserved).

---

## **2. Context: Normal Injection vs Lazy Injection**

### **Without `@Lazy`**

```java
@Autowired
public MyService(MyBean myBean) {
    this.myBean = myBean;
}
```

- **Spring at startup**:
    1. Finds `MyService` bean and sees it depends on `MyBean`.
    2. Immediately creates an instance of `MyBean`.
    3. Injects the fully initialized `MyBean` into `MyService`.
- **Result**: Both `MyService` and `MyBean` are created eagerly at startup.

---
### **With `@Lazy`**

```java
@Autowired
public MyService(@Lazy MyBean myBean) {
    this.myBean = myBean;
}
```

- **Spring at startup**:
    1. Finds `MyService` bean and sees it depends on `MyBean`.
    2. Instead of creating `MyBean` immediately, Spring **creates a proxy** object.
        - This proxy looks like a `MyBean` to `MyService`.
        - No real `MyBean` instance is created yet.
    3. Injects the proxy into `MyService`.
    
- **When `MyService` calls a method on `myBean` for the first time**:
    1. The proxy intercepts the call.
    2. Spring **creates the real `MyBean` instance** at this moment.
    3. The proxy forwards the method call to the real bean.
    
- **Result**: `MyBean` is created **only when needed**, not at application startup.

---

## **3. Flow Diagram (Conceptual)**

```
ApplicationContext starts
       │
       ▼
Spring detects MyService bean
       │
       ▼
Constructor parameter: @Lazy MyBean
       │
       ├─> Spring creates a proxy instead of MyBean
       │
       ▼
MyService instance is created with proxy
       │
       ▼
Method call on myBean (proxy) happens
       │
       ├─> Proxy triggers creation of real MyBean
       │
       └─> Real MyBean instance is initialized
       │
       ▼
Method call is executed on real MyBean
```

---

## **4. Practical Example**

```java
@Component
class MyBean {
    public MyBean() {
        System.out.println("MyBean is created!");
    }

    public void doWork() {
        System.out.println("Doing work in MyBean");
    }
}

@Component
class MyService {
    private final MyBean myBean;

    @Autowired
    public MyService(@Lazy MyBean myBean) {
        System.out.println("MyService is created!");
        this.myBean = myBean;
    }

    public void process() {
        myBean.doWork(); // MyBean is created here if not already
    }
}

@SpringBootApplication
public class DemoApplication {
    public static void main(String[] args) {
        var context = SpringApplication.run(DemoApplication.class, args);
        var service = context.getBean(MyService.class);
        System.out.println("Calling process() on MyService");
        service.process(); // MyBean is created now
    }
}
```

**Output:**

```
MyService is created!
Calling process() on MyService
MyBean is created!
Doing work in MyBean
```


## 5. **Scenario: Circular Dependency**

Suppose we have two beans that depend on each other:

```java
@Component
class ServiceA {
    private final ServiceB serviceB;

    @Autowired
    public ServiceA(@Lazy ServiceB serviceB) {
        this.serviceB = serviceB;
        System.out.println("ServiceA created");
    }

    public void callB() {
        serviceB.doSomething();
    }
}

@Component
class ServiceB {
    private final ServiceA serviceA;

    @Autowired
    public ServiceB(ServiceA serviceA) {
        this.serviceA = serviceA;
        System.out.println("ServiceB created");
    }

    public void doSomething() {
        System.out.println("ServiceB method called");
    }
}
```

---

### **1. Without `@Lazy`**
- Spring tries to create `ServiceA`.
- `ServiceA` needs `ServiceB` → Spring tries to create `ServiceB`.
- `ServiceB` needs `ServiceA` → circular dependency detected.
- Spring throws **`BeanCurrentlyInCreationException`**.

✅ Constructor injection cannot resolve circular dependencies unless `@Lazy` is used.

---
### **2. With `@Lazy` on `ServiceB` in `ServiceA`**

```java
@Autowired
public ServiceA(@Lazy ServiceB serviceB) { ... }
```

#### **Flow*
1. Spring starts creating `ServiceA`.
2. Sees `@Lazy ServiceB`:
    - Instead of creating `ServiceB` immediately, Spring **injects a proxy** for `ServiceB` into `ServiceA`.
3. `ServiceA` creation completes successfully.
4. Spring creates `ServiceB`:
    - `ServiceB` depends on `ServiceA`.
    - `ServiceA` already exists, so it is injected into `ServiceB`.
5. `ServiceB` creation completes successfully.
6. Circular dependency resolved without errors.

---

### **3. Proxy Behavior**
- `ServiceA` holds a **proxy** reference to `ServiceB`.
- Actual `ServiceB` instance is created **when a method on the proxy is first called**, but in practice Spring will create the real `ServiceB` immediately afterward because it’s a singleton.
- The key is that **`@Lazy` breaks the circular creation chain**, allowing Spring to finish one bean before creating the other.

---
### **4. Flow Diagram (Simplified)**

```
Start creating ServiceA
        │
        ├─> Needs ServiceB (@Lazy)
        │       └─> Spring injects a proxy instead of real ServiceB
        │
ServiceA created successfully
        │
Start creating ServiceB
        ├─> Needs ServiceA
        │       └─> ServiceA already exists → inject it
        │
ServiceB created successfully
        │
Circular dependency resolved!
```

---
### **5. Output if we call a method**

```java
ApplicationContext context = SpringApplication.run(DemoApplication.class);
ServiceA a = context.getBean(ServiceA.class);
a.callB();
```

**Console Output:**
```
ServiceA created
ServiceB created
ServiceB method called
```

---
### **Key Points**
- **`@Lazy` on one side of the circular dependency** allows Spring to inject a proxy and break the cycle.
- Circular dependency resolution **works only with setter/field injection or lazy proxies for constructors**.
- Constructor injection **without `@Lazy` will fail** for circular dependencies.
