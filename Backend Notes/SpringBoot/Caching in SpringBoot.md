
## 1. What is Caching?

- **Caching** is storing **frequently accessed data in memory** (or fast storage) so that repeated requests **don’t hit the database or slow computations**.
    
- Improves **performance**, reduces **latency**, and **reduces load** on services.
    

---

## 2. Why Use Caching?

|Benefit|Explanation|
|---|---|
|Faster response|Data is retrieved from memory instead of DB or API|
|Reduced database load|Less queries → lower CPU/memory usage on DB|
|Cost efficiency|Avoids repeated expensive computations or network calls|
|Scalability|Handles more requests without adding DB resources|

---

## 3️. How Spring Boot Supports Caching

- Spring Boot provides **built-in caching abstraction** via `spring-boot-starter-cache`.
- Supports multiple **cache providers** like:
    - **ConcurrentMapCache** (default, in-memory)
    - **EhCache**, **Caffeine**, **Redis**, **Hazelcast**, etc.
- Spring manages **cache lifecycle** via **annotations** and **AOP proxies**.


---

## 4️. Enabling Caching in Spring Boot

```java
@SpringBootApplication
@EnableCaching
public class CacheDemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(CacheDemoApplication.class, args);
    }
}
```

- `@EnableCaching` activates Spring’s cache abstraction.
- Spring will automatically **create proxies** for beans with cache annotations.

---

## 5️. Cache Annotations

### a) `@Cacheable`

- **Definition:** Cache method result. If data exists in cache, return it instead of executing the method.

```java
@Service
public class UserService {
    
    @Cacheable(value = "users", key = "#id")
    public User getUserById(Long id) {
        simulateSlowService();
        return userRepository.findById(id).orElse(null);
    }

    private void simulateSlowService() {
        try { Thread.sleep(3000); } catch (InterruptedException e) {}
    }
}
```

- `value` = cache name
- `key` = SpEL expression to generate cache key


**Flow:**
1. Check if cache `"users"` has key `id`.
2. If yes → return cached value.
3. If no → call method, store result in cache.

---

### b) `@CachePut`
- **Definition:** Updates the cache **without skipping method execution**.
- Useful when you modify data and want cache updated.

```java
@CachePut(value = "users", key = "#user.id")
public User updateUser(User user) {
    return userRepository.save(user);
}
```

- Method always runs → cache updated with the returned value.    

---

### c) `@CacheEvict`
- **Definition:** Removes entries from cache.
- Useful when data is deleted or invalidated.

```java
@CacheEvict(value = "users", key = "#id")
public void deleteUser(Long id) {
    userRepository.deleteById(id);
}
```

- Optional: `allEntries = true` → clear the whole cache.    

---
### d) `@Caching`
- Combine multiple cache operations.

```java
@Caching(
    put = { @CachePut(value = "users", key = "#user.id") },
    evict = { @CacheEvict(value = "allUsers", allEntries = true) }
)
public User saveUser(User user) {
    return userRepository.save(user);
}
```

---

## 6️. Default Cache Implementation
- If **no external cache** is configured, Spring Boot uses **ConcurrentMapCache**:
    - Simple in-memory **HashMap** based cache.
    - **Not suitable for distributed systems**.
- For production, use **Redis, EhCache, Caffeine, or Hazelcast**.    

---
## 7️. Example: Redis Cache Integration

```java
@SpringBootApplication
@EnableCaching
public class CacheApp { }

@Configuration
public class RedisConfig {
    @Bean
    public RedisCacheManager cacheManager(RedisConnectionFactory factory) {
        return RedisCacheManager.builder(factory).build();
    }
}

@Service
public class ProductService {
    @Cacheable(value = "products", key = "#id")
    public Product getProduct(Long id) {
        return productRepository.findById(id).orElse(null);
    }
}
```

- Redis is **distributed**, persists data across multiple instances, and is ideal for **scalable apps**.

---
## 8️. Cache Key & TTL (Time-to-Live)

- **Key**: Identifies cached data, usually generated using SpEL (`#id`, `#root.methodName`).
- **TTL**: How long data stays in cache (depends on provider, e.g., Redis `expire`, Caffeine `expireAfterWrite`).

---

## 9️. Best Practices

|Practice|Reason / Explanation|
|---|---|
|Keep beans stateless|Spring caching uses proxies; thread safety is important|
|Use meaningful cache keys|Avoid collisions, ensure correctness|
|Use TTL for volatile data|Prevent stale cache entries|
|Avoid caching large objects|Can increase memory usage|
|Combine with async if needed|Long-running computations → cache results asynchronously|

---

## 10️. Lifecycle Overview

```text
[Client Request]
       │
       ▼
[Service Method with @Cacheable]
       │
       ├─ Check cache
       │    ├─ Hit → return cached result
       │    └─ Miss → execute method
       │
       ▼
Store result in cache
       │
       ▼
Return response to client
```

---

✅ **Key Points**

- Spring caching is **declarative** via annotations.
- Works **out-of-the-box** with in-memory cache; supports **distributed caches** for production.
- Proper **key design and cache invalidation** are crucial for correctness.
- Supports **combined annotations** (`@Caching`) and **asynchronous caching**.
