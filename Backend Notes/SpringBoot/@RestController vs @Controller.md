
# 1️. Why Do We Have Two Different Controllers?
Spring MVC supports two types of applications:
1. **Traditional MVC (Server-side rendering)**
    - Returns views (JSP, Thymeleaf, etc.)
2. **REST APIs (JSON/XML responses)**
    - Returns data, not views

To support both cleanly, Spring provides:
- `@Controller`
- `@RestController`

---
# 2️. `@Controller`

## What It Is
A specialization of `@Component` used in **Spring MVC applications** to return **views**.
```java
@Controller
public class UserController {
}
```

---

##  Default Behavior
- Returns **view name**
- Uses `ViewResolver`
- Does NOT automatically serialize objects to JSON

---
##  Example (Traditional MVC)
```java
@Controller
public class UserController {

    @GetMapping("/home")
    public String home() {
        return "home";   // resolves to home.jsp or home.html
    }
}
```

Spring:
1. Receives request
2. Calls method
3. Returns `"home"`
4. ViewResolver resolves view
5. Renders HTML page

---
##  Returning JSON with `@Controller`
You must explicitly add:
```java
@ResponseBody
```

Example:
```java
@Controller
public class UserController {

    @GetMapping("/user")
    @ResponseBody
    public User getUser() {
        return new User("John");
    }
}
```

Now Spring converts object → JSON.

---
# 3️. `@RestController`

## What It Is
`@RestController` is a shortcut for:
```java
@Controller + @ResponseBody
```

It is used for building **REST APIs**.
```java
@RestController
public class UserController {
}
```

---
##  Default Behavior
- Returns **data**
- Automatically serializes object → JSON/XML
- Uses `HttpMessageConverter`

No need for `@ResponseBody` on each method.

---
##  Example

```java
@RestController
public class UserController {

    @GetMapping("/user")
    public User getUser() {
        return new User("John");
    }
}
```

Spring:
1. Calls method
2. Gets object
3. Uses `MappingJackson2HttpMessageConverter`
4. Returns JSON response

---
# 4️. Internal Difference

### `@Controller`
Return type handling:
- If String → treated as view name
- If object + `@ResponseBody` → JSON

Uses:
```
ViewResolver
```

---

### `@RestController`
Return type handling:
- Always treated as response body
- Uses:
```
HttpMessageConverter
```

---
# 5️. Core Differences Table

| Feature                   | `@Controller`             | `@RestController` |
| ------------------------- | ------------------------- | ----------------- |
| Primary Use               | MVC (views)               | REST APIs         |
| Returns                   | View name                 | JSON/XML          |
| Needs `@ResponseBody`?    | Yes                       | No                |
| Uses ViewResolver         | ✅ Yes                     | ❌ No              |
| Uses HttpMessageConverter | Only with `@ResponseBody` | Always            |
| Introduced In             | Spring MVC                | Spring 4          |

---
# 6️. When to Use What?

###  Use `@Controller` When:
- Building server-rendered apps
- Using JSP / Thymeleaf
- Returning HTML pages

---
###  Use `@RestController` When:
- Building REST APIs
- Returning JSON
- Backend for frontend/mobile apps

---
# 7️. Common Interview Traps

---

###  1. What happens if you forget `@ResponseBody` in REST app?

Spring will try to resolve view name → leads to error like:

```
404 view not found
```

---

###  2. Does `@RestController` bypass DispatcherServlet?
No.
Both go through:
```
DispatcherServlet → HandlerMapping → HandlerAdapter
```
Only difference is return value handling.

---
# 8️. Flow Comparison

### `@Controller`
```
Request → Controller → View Name → ViewResolver → HTML Response
```

---
### `@RestController`
```
Request → Controller → Object → HttpMessageConverter → JSON Response
```
