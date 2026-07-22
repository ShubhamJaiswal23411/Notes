
Spring Boot (Spring Framework) defines **bean scopes** that control the lifecycle and visibility of beans in the **Spring IoC container**.

## 1. **Singleton (Default)**

- **Definition:** Single instance per Spring container.
- **Usage:** Stateless beans, services, repositories.
- **Example:**
    

```java
@Service
public class UserService { }
```

- **Lifecycle:** Created once at container startup.
    

---

## 2. **Prototype**

- **Definition:** New instance every time the bean is requested.
- **Usage:** Stateful beans, objects that need fresh data per request.
- **Example:**
    

```java
@Component
@Scope("prototype")
public class Task { }
```

- **Lifecycle:** Spring container creates a new instance on each injection or `getBean()` call.
    
---

## 3. **Request**

- **Definition:** One instance per HTTP request.
- **Usage:** Web applications, per-request data handling.
- **Example:**
    

```java
@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class RequestBean { }
```

- **Lifecycle:** Created at request start, destroyed at request end.

---

## 4. **Session**

- **Definition:** One instance per HTTP session.
- **Usage:** Store user session data, login info, shopping cart.
- **Example:**

```java
@Component
@Scope(value = "session", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class SessionBean { }
```

- **Lifecycle:** Created on session creation, destroyed on session timeout/termination.

---

## 5. **Application**

- **Definition:** One instance per ServletContext (per web application).
- **Usage:** Shared app-wide beans across multiple sessions/requests.
- **Example:**

```java
@Component
@Scope(value = "application", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class AppBean { }
```

---

## 6. **Websocket**

- **Definition:** One instance per WebSocket session.    
- **Usage:** Real-time apps with WebSocket sessions.
- **Example:**

```java
@Component
@Scope(value = "websocket", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class WebSocketBean { }
```

---

### ⚡ Quick Summary Table

| Scope       | Instance per      | Use case                        |
| ----------- | ----------------- | ------------------------------- |
| Singleton   | Container         | Stateless services              |
| Prototype   | Each request      | Stateful beans                  |
| Request     | HTTP request      | Per-request data                |
| Session     | HTTP session      | Session-specific data           |
| Application | ServletContext    | App-wide shared data            |
| WebSocket   | WebSocket session | Real-time user connection state |

---


# Spring Bean Scoped Proxy (`proxyMode = ScopedProxyMode.TARGET_CLASS`)

## Why It’s Required

Spring beans like `request`, `session`, or `websocket` are **not singleton**. That means:

- Their lifecycle is shorter than the singleton beans that depend on them.
- If a **singleton bean injects a shorter-lived bean**, Spring cannot directly inject it because the singleton is created **once**, but the short-lived bean may **not exist yet** at that time.
    

💡 **Problem:**

```java
@Service
public class UserService {
    @Autowired
    private RequestBean requestBean; // Request bean injected into singleton
}
```

- The `RequestBean` only exists **per HTTP request**, but `UserService` is a singleton.
- Without a proxy, Spring will throw **BeanNotOfRequiredTypeException** or inject the wrong instance.

---

## What `proxyMode = ScopedProxyMode.TARGET_CLASS` Does

1. Spring creates a **proxy object** instead of the actual bean.
2. The proxy acts as a **placeholder** in singleton beans.
3. When a method on the proxy is called, Spring **fetches the correct scoped instance** at runtime.

```java
@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class RequestBean { }
```

- `TARGET_CLASS` → Uses **CGLIB subclass proxy** (class-based).
- Alternative: `INTERFACES` → Uses **JDK dynamic proxy** (interface-based).

✅ **Effect:** Singleton beans can safely use request/session-scoped beans without worrying about lifecycle mismatches.

---

# Spring Scoped Proxy Diagram

```text
[Singleton Bean]
       │
       │ injects
       ▼
   [Proxy Object]
       │
       │ at runtime
       ▼
[Actual Scoped Bean]
(Request / Session / Application / WebSocket)
```

# Important Point about Application Scoped Beans:


## Spring Boot ApplicationContext vs ServletContext Startup Flow

### 1. Key Concepts

|Term|What it is|
|---|---|
|**ServletContext**|Part of the Java EE / Servlet API. Represents the **entire web app** in the servlet container (like Tomcat). Lives as long as the web app is deployed.|
|**ApplicationContext**|Spring’s IoC container. Manages **Spring beans** and their lifecycles. Can be tied to **ServletContext** in web apps (`WebApplicationContext`). Lives as long as Spring runs.|
|**Tomcat / Servlet Container**|JVM process that runs web apps. Can host multiple web apps, each with its own `ServletContext`.|

💡 Note: ApplicationContext runs **inside the JVM**, but it’s **initialized when the web app starts**, using the ServletContext as its environment.

---

### 2. Startup Flow (Spring Boot with embedded Tomcat)

```text
JVM starts
   │
   │ Spring Boot main()
   ▼
Embedded Tomcat starts
   │
   │ Servlet container initialized
   ▼
ServletContext created (represents this web app)
   │
   │ Spring Boot creates WebApplicationContext
   ▼
ApplicationContext initialized
   │
   │ All Spring beans are instantiated
   │ Singleton beans created immediately
   │ Application-scoped beans tied to ServletContext
   ▼
Web app ready to handle requests
```

#### Key Points

1. **ServletContext first:**
    - Embedded Tomcat (or external container) starts the web app → creates `ServletContext`.
    - `ServletContext` is provided to Spring Boot to initialize the web app.
        
2. **ApplicationContext second:**
    - Spring Boot starts `SpringApplication.run()`.
    - It creates a `WebApplicationContext`, **binding it to the ServletContext**.
    - All Spring beans are instantiated according to their scopes.
        
3. **Bean lifecycles:**
    - **Singleton beans** → created once at context startup.
    - **Application beans** → tied to `ServletContext`, shared across all requests/sessions.
    - **Request/session beans** → created per HTTP request or session, managed via proxies.
        

---

### 3. Visual Analogy

```text
[ JVM ]
   │
   │ Embedded Tomcat starts
   ▼
[ Servlet Container ]
   │
   │ Web app deployed
   ▼
[ ServletContext ]
   │
   │ Spring initializes
   ▼
[ WebApplicationContext (Spring IoC) ]
   │
   │ Manages beans
   ▼
[ Singleton | Application | Session | Request beans ]
```

- JVM runs Tomcat → Tomcat runs ServletContext → ServletContext **environment** initializes Spring ApplicationContext → Spring manages beans within this context.
    

---

### ⚡ Summary

- **JVM**: process hosting Tomcat and Spring.
- **ServletContext**: represents the web app in Tomcat, exists as long as app is deployed.
- **ApplicationContext**: Spring’s IoC container, manages beans, tied to ServletContext in web apps.
- **Application-scoped beans**: live as long as ServletContext (app life), shared across requests & sessions.
