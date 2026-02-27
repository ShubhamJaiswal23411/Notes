
# 1️. Excluding Auto-Configuration

Used when you want to completely disable a built-in auto-config.

### Example

```java
@SpringBootApplication(
    exclude = DataSourceAutoConfiguration.class
)
public class App { }
```

Or in properties:

```properties
spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
```

# 2️. `@ConditionalOnMissingBean`

Spring Boot auto-configurations usually use this.

👉 If **you define your own bean**, Boot backs off.

### Example

```java
@Configuration
public class MyConfig {

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate(); // overrides Boot default
    }
}
```

### Is `@Configuration` needed?

✅ Yes — because you are defining beans.  

---

# 3️. Using `application.properties`

Most auto-config is property-driven.

### Example

```properties
server.port=9090
spring.datasource.url=jdbc:mysql://localhost:3306/test
```

Spring Boot reads these and configures beans accordingly.

---

# 4️. `@ConditionalOnProperty`

Create configuration that loads only when a property is set.
### Example

```java
@Configuration
@ConditionalOnProperty(
    name = "feature.enabled",
    havingValue = "true"
)
public class FeatureConfig {

    @Bean
    public MyService myService() {
        return new MyService();
    }
}
```

```properties
feature.enabled=true
```

### Is `@Configuration` needed?

✅ Yes — because you're defining beans conditionally.

---
# 🔎 Quick Summary

|Feature|Needs `@Configuration`?|
|---|---|
|Exclude auto-config|❌ No|
|`@ConditionalOnMissingBean` (custom bean)|✅ Yes|
|`application.properties`|❌ No|
|`@ConditionalOnProperty`|✅ Yes|
