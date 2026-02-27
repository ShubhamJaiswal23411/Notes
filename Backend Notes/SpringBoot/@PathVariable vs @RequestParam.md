
# 1️. Why Do We Need Them?

When handling HTTP requests in Spring MVC, we often need to extract data from the URL.
Example requests:
```
GET /users/10
GET /users?id=10
```

Both send `10`, but in different ways.
Spring provides two annotations to bind these values to method parameters
- `@PathVariable`
- `@RequestParam`

---
# 2️. What is `@PathVariable`?

`@PathVariable` extracts values from the **URI path itself**.
Example URL:
```
GET /users/10
```

Controller:
```java
@GetMapping("/users/{id}")
public User getUser(@PathVariable int id) {
    return service.findById(id);
}
```

Here:
- `{id}` is part of URL path
- `10` is bound to `id`

---

##  Key Characteristics
- Value is part of the URL structure
- Used for identifying specific resources
- Mandatory by default
- Clean RESTful design

---

# 3️. What is `@RequestParam`?

`@RequestParam` extracts values from **query parameters**.
Example URL:
```
GET /users?id=10
```

Controller:
```java
@GetMapping("/users")
public User getUser(@RequestParam int id) {
    return service.findById(id);
}
```

Here:
- `id=10` comes from query string

---

##  Key Characteristics
- Used for filtering, searching, optional data
- Optional by default (can set required = true/false)
- Supports default values

---

##  Optional Example
```java
@GetMapping("/users")
public List<User> getUsers(
    @RequestParam(required = false) String type) {
}
```

---

# 4️. Conceptual Difference

| Concept      | PathVariable            | RequestParam          |
| ------------ | ----------------------- | --------------------- |
| Part of URL  | Yes                     | No                    |
| Query String | No                      | Yes                   |
| Used For     | Resource identification | Filtering / options   |
| RESTful      | More RESTful            | Less REST-focused     |
| Optional?    | Usually No              | Yes (can be optional) |

---
# 5. When to Use What?

###  Use `@PathVariable` When:
- Identifying a specific resource
- REST API design
- Mandatory data
- Hierarchical structure

Example:
```
/users/10/orders/5
```

---

###  Use `@RequestParam` When:
- Filtering
- Searching
- Pagination
- Optional parameters

Example:

```
/users?page=1&size=10&sort=name
```

---

# 7️. Advanced Differences

---

## 1. Default Value Support
Only `@RequestParam` supports:
```java
@RequestParam(defaultValue = "1") int page
```

`@PathVariable` does not support default values.

---

## 2. Optional Handling
`@RequestParam`:
```java
@RequestParam(required = false)
```

`@PathVariable`:
- Required by default
- Can use `required = false` in newer versions, but not typical

---

# 8. Tricky Interview Questions

---

###  1.  What happens if path variable missing?

→ 404 error (URL doesn’t match mapping)

---

###  2. What happens if required request param missing?

→ 400 Bad Request (unless required = false)

---

### 3. Which is more RESTful?

`@PathVariable` because it is asking for resource which is one the principles of rest architecture it is based on resources.

---

# 9. Quick Comparison Table

| Feature               | `@PathVariable` | `@RequestParam`           |
| --------------------- | --------------- | ------------------------- |
| Extracts From         | URL path        | Query string              |
| Mandatory             | Yes (default)   | Yes/No                    |
| Default Value         | ❌ No            | ✅ Yes                     |
| Best For              | Resource ID     | Filters / optional params |
| Example               | `/users/10`     | `/users?id=10`            |
| HTTP Error if Missing | 404             | 400                       |
