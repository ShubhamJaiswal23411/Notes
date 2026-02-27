
(Important: `MultipartFile` is not an annotation — it’s a type used for file uploads. It usually works with `@RequestParam` or `@RequestPart`.)

---
# 1️. Why Do We Need These?
When a client sends data to a Spring MVC controller, the data can come in different formats:
- JSON (REST APIs)
- Form data (`application/x-www-form-urlencoded`)
- Multipart form data (file uploads)
Spring provides different mechanisms to bind incoming request data to method parameters.

---
# 2️. `@RequestBody`

##  What It Is
`@RequestBody` binds the **HTTP request body** to a Java object.
Used mainly in **REST APIs**.

---
##  When Is It Used?
- Content-Type: `application/json`
- Content-Type: `application/xml`
Example request:
```json
POST /users
Content-Type: application/json

{
  "name": "John",
  "age": 25
}
```

Controller:
```java
@PostMapping("/users")
public User createUser(@RequestBody User user) {
    return userService.save(user);
}
```

---
##  How It Works Internally

Spring uses:
```
HttpMessageConverter
```

Example:
- `MappingJackson2HttpMessageConverter` → JSON → Java object

Flow:
1. Reads request body
2. Converts JSON → Java object
3. Injects into parameter

---
##  Key Characteristics
- Reads raw request body
- Uses message converters
- Typically used in REST controllers
- Cannot be used with form data directly

---
# 3️. `@ModelAttribute`

## What It Is
`@ModelAttribute` binds **form data or query parameters** to a Java object.
Used mainly in **traditional Spring MVC (form submissions)**.

---
##  Example (Form Submission)
HTML form:
```html
<form action="/users" method="post">
    <input name="name"/>
    <input name="age"/>
</form>
```

Controller:
```java
@PostMapping("/users")
public String saveUser(@ModelAttribute User user) {
    return "success";
}
```

---
##  How It Works Internally

Spring:
1. Creates empty object
2. Uses reflection
3. Matches request parameters to object fields
4. Calls setters

No JSON parsing involved.

---
##  Default Behavior
If no annotation is present:
- Complex types → treated as `@ModelAttribute`
- Simple types → treated as `@RequestParam`

---

##  Key Characteristics
- Works with form data
- Works with query params
- Does NOT use HttpMessageConverter
- Used in MVC apps with views (JSP/Thymeleaf)

---

# 4️. `MultipartFile` (File Upload Handling)

## What It Is
`MultipartFile` is an interface used to handle **file uploads**.
Works with:
- `multipart/form-data`
- `@RequestParam`
- `@RequestPart`

---
##  Example

HTML:
```html
<form method="post" enctype="multipart/form-data">
    <input type="file" name="file"/>
</form>
```

Controller:
```java
@PostMapping("/upload")
public String uploadFile(@RequestParam("file") MultipartFile file) {
    System.out.println(file.getOriginalFilename());
    return "uploaded";
}
```

---
##  How It Works Internally

Spring uses:
```
MultipartResolver
```

It:
1. Detects multipart request
2. Parses request
3. Extracts file
4. Wraps file inside `MultipartFile`

---
# 5️. `@RequestPart` (Advanced Multipart Use Case)

Used when:
- Sending JSON + file together
- Content-Type: multipart/form-data

Example:
```java
@PostMapping("/upload")
public String upload(
    @RequestPart("user") User user,
    @RequestPart("file") MultipartFile file) {
}
```

Here:
- JSON part → converted using `HttpMessageConverter`
- File part → converted to `MultipartFile`

---

# 6️. Core Differences

| Feature               | `@RequestBody`    | `@ModelAttribute`             | `MultipartFile`          |
| --------------------- | ----------------- | ----------------------------- | ------------------------ |
| Data Source           | Request body      | Form fields / query params    | File upload              |
| Content-Type          | JSON/XML          | form-urlencoded               | multipart/form-data      |
| Uses MessageConverter | ✅ Yes             | ❌ No                          | ❌ No                     |
| Object Creation       | Direct conversion | Create empty object + setters | File wrapper             |
| REST API Usage        | ✅ Yes             | ❌ Rare                        | ✅ Yes (for file uploads) |
| Traditional MVC       | ❌ No              | ✅ Yes                         | ✅ Yes                    |

---

# 7️. When to Use What?

###  Use `@RequestBody` When:
- Building REST APIs
- Receiving JSON payload
- Client = frontend/mobile/Postman

---
###  Use `@ModelAttribute` When:
- Handling HTML form submissions
- Traditional MVC with views
- Form-based applications

---
###  Use `MultipartFile` When:
- Uploading files
- Handling images/document
- Using multipart/form-data

---
# 8️. Common Mistakes (Interview Traps)

---

###  1. Can we use `@RequestBody` with form data?
No. It expects full request body like JSON.
###  2. Can we use `@ModelAttribute` for JSON?
Not properly. JSON requires `HttpMessageConverter`.
###  3. Why does `@RequestBody` fail with file upload?
Because file upload uses multipart encoding, not raw JSON body.
###  4. Can we combine file + JSON?
Yes, using:
```java
@RequestPart
```

---

# 9️. Internal Resolver Used

All are handled via:
```
HandlerMethodArgumentResolver
```

Specifically:
- `RequestResponseBodyMethodProcessor`
- `ModelAttributeMethodProcessor`
- `RequestPartMethodArgumentResolver`

# 🏁 Final Takeaway
- JSON → `@RequestBody`
- Form Data → `@ModelAttribute`
- File Upload → `MultipartFile`
- JSON + File → `@RequestPart`