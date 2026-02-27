
## 1. **Client Sends HTTP Request**

- Browser, Postman, mobile app, or another service sends a request:
    
    ```
    GET /api/users/123
    Host: localhost:8080
    ```
    

---

## 2. **Servlet Container (Tomcat / Jetty) Receives Request**

- Embedded **Tomcat** in Spring Boot handles incoming HTTP requests.
    
- Creates **`HttpServletRequest`** and **`HttpServletResponse`** objects.
    
- Passes request to **DispatcherServlet**.
    

```text
[Client] ---> [Tomcat] ---> [HttpServletRequest/Response] ---> [DispatcherServlet]
```

---

## 3. **DispatcherServlet**

- Central **front controller** in Spring MVC.
    
- Responsibilities:
    
    1. Determine which **controller** should handle the request (via HandlerMapping).
        
    2. Prepare **handler execution chain** (controller + interceptors).
        
    3. Manage request **lifecycle** (pre-processing, post-processing).
        

```text
DispatcherServlet
  ├─ HandlerMapping (finds controller method)
  ├─ Interceptors (preHandle)
  └─ Invokes Controller method
```

---

## 4. **HandlerMapping & Controller Selection**

- Spring looks at **`@RequestMapping` / `@GetMapping` / `@PostMapping` annotations**.
    
- Matches URL, HTTP method, headers, etc., to the correct controller method.
    

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.getUserById(id);
    }
}
```

- Example: `/api/users/123` → `getUser(123)`.
    

---

## 5. **Controller Method Execution**

- DispatcherServlet **injects dependencies** (like `UserService`) via **ApplicationContext**.
    
- Controller method executes **business logic**:
    
    1. Calls service layer → interacts with database via repository.
        
    2. Can perform validation, caching, transactions, etc.
        

```text
UserController.getUser()
  └─ UserService.getUserById()
      └─ UserRepository.findById(123)
```

---

## 6. **Return Value / Response Handling**

- Controller returns a value (POJO, List, Map, etc.)
    
- Spring uses **HttpMessageConverters** to serialize the object (usually JSON or XML).
    
- Response body is written to **HttpServletResponse**.
    

```text
Controller returns User object
  └─ Jackson (ObjectMapper) serializes to JSON
      └─ Response sent via HttpServletResponse to client
```

---

## 7. **Interceptors / Filters / Exception Handling**

- Before sending response:
    
    - **Filters**: pre/post-processing (security, logging, CORS)
        
    - **Interceptors**: preHandle / postHandle / afterCompletion
        
    - **Exception handlers**: `@ControllerAdvice` for global exception handling
        

```text
[DispatcherServlet]
  └─ Interceptors & Filters
      └─ GlobalExceptionHandler (if error)
```

---

## 8. **Client Receives Response**

- HTTP status code + headers + serialized body returned to client:
    

```json
{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com"
}
```

- Browser / Postman renders or processes the response.
    

---

## 9. **Summary Flow (Visual)**

```text
[Client]
   │
   ▼
[Embedded Tomcat / Servlet Container]
   │
   ▼
[DispatcherServlet (Front Controller)]
   │
   ├─ Filters (CORS, Security)
   ├─ HandlerMapping (find controller)
   ├─ Interceptors (preHandle)
   ▼
[Controller Method] ----> [Service Layer] ----> [Repository / DB]
   │
   ▼
Return Object
   │
   ├─ HttpMessageConverter (JSON/XML)
   ├─ Interceptors (postHandle)
   └─ Filters (post-processing)
   │
   ▼
[HttpServletResponse] ---> [Client]
```

---

💡 **Key Notes**

- **DispatcherServlet** is the heart of Spring MVC.
    
- **ApplicationContext** provides all the beans (controllers, services, repositories).
    
- **Filters** are servlet-level (Tomcat handles them first).
    
- **Interceptors** are Spring MVC-level (after DispatcherServlet gets the request).
    
- **Proxies** (like `@Scope("request")`) ensure the correct scoped bean is used per request.

# Spring Boot Request Handling After Tomcat Receives a Request

## 1. **Does DispatcherServlet exist in Spring Boot?**

✅ Yes.

- Spring Boot **auto-configures `DispatcherServlet`** if Spring MVC (`spring-boot-starter-web`) is on the classpath.
    
- `DispatcherServlet` is the **front controller** for all HTTP requests.
    
- It is **singleton** per `ServletContext` (one instance for the entire web app).
    
- All requests go through this single DispatcherServlet instance.
    

---

## 2. **Threading Model in Tomcat**

### a) Tomcat Thread Pool

- Embedded Tomcat uses a **thread pool** (`Executor`) to handle incoming requests.
    
- **Default:** `maxThreads = 200` (can be configured in `application.properties`).
    
- **Request flow:**
    

```text
[Client Request]
       │
       ▼
   Tomcat Acceptor Thread
       │ (assigns request to worker thread from pool)
       ▼
   Worker Thread
       │
       └─ Handles request via Servlet (DispatcherServlet)
```

- Each request is handled by a **separate thread from the pool**.
    
- If there are **thousands of concurrent requests** but only 200 threads:
    
    - Requests beyond the max pool size are **queued** (or rejected if queue full).
        

---

### b) Key Tomcat Threads

|Thread Type|Responsibility|
|---|---|
|**Acceptor**|Listens on port (8080), accepts TCP connection|
|**Poller / Selector**|Monitors sockets for read/write events|
|**Worker Threads**|Executes the Servlet’s `service()` method (DispatcherServlet)|

---

## 3. **Request to DispatcherServlet**

- **DispatcherServlet is singleton**, but it is **thread-safe**.
    
- Tomcat worker thread calls `DispatcherServlet.service(request, response)` for **each request**.
    

```text
Tomcat Worker Thread
       │
       ▼
DispatcherServlet.service(request, response)
       │
       ▼
HandlerMapping → Controller → Service → Repository
```

- Each request is independent; shared beans (singleton) must be **stateless** or thread-safe.
    
- **Request/Session scoped beans** are **proxied** to ensure thread-safety.
    

---

## 4. **How Thousands of Requests Are Managed**

1. **Thread-per-request model:**
    
    - Each request gets a **worker thread** from Tomcat thread pool.
        
    - DispatcherServlet handles the request on that thread.
        
2. **Concurrency limitations:**
    
    - Max threads = 200 (default).
        
    - Excess requests wait in **queue** (backlog) until a thread is free.
        
    - Can be tuned via `server.tomcat.max-threads` in `application.properties`.
        
3. **Thread safety concerns:**
    
    - **Singleton beans** → must be stateless or synchronized.
        
    - **Scoped beans** → request/session scoped beans use proxies to maintain per-thread/request instances.


---

## 5. **Async Requests (Optional)**

- Spring Boot supports **asynchronous processing**:
    
    - `@Async` on service methods → runs in separate thread pool.
        
    - `DeferredResult`, `WebAsyncTask` → allows releasing Tomcat worker threads for high concurrency.
        

```text
[DispatcherServlet]
       │
       └─ If @Async / WebAsyncTask → hands off to separate thread pool
       │
       └─ Original worker thread free to handle next request
```

---

## 6. **Summary of Flow After Tomcat Receives Request**

```text
[Tomcat Acceptor] --> accepts TCP connection
       │
       ▼
[Worker Thread from Pool] --> calls DispatcherServlet.service(request, response)
       │
       ▼
[DispatcherServlet (singleton)]
       │
       ├─ Filters
       ├─ HandlerMapping → finds Controller
       ├─ Interceptors → preHandle
       ▼
[Controller Method]
       │
       └─ Service → Repository → DB
       ▼
Return Object
       │
       └─ HttpMessageConverter → serialize JSON/XML
       ▼
Worker thread writes to HttpServletResponse
```

- **DispatcherServlet exists once per web app** but is thread-safe → handles all requests concurrently on multiple worker threads.
    
- **Thread pool limits** maximum concurrency. Requests beyond that queue up.
- We can have multiple DispatcherServlet and each DispatcherServlet will have its own application context. but all DispatcherServlet will be tied to one single ServletContext.
    

---

💡 **Analogy**

- DispatcherServlet = **single receptionist** at an office.
    
- Worker threads = **staff who process requests**.
    
- Thousands of visitors = queued if all staff are busy.
    
- Request-scoped beans = **temporary notebook for each visitor** → safe to use concurrently
	
- **DispatcherServle**t is actually connected to servlet context instead of application context instead if you define multiple DispatcherServlet then each will have its own application context, we can do this in case there are versioned apis and we want to create a distinction on which dispatcher will handle which version of the apis.
