
Creating a **custom scope in Spring Boot** means defining your own bean lifecycle beyond the standard scopes (`singleton`, `prototype`, `request`, etc.).

Example: a scope where one bean instance is created per **thread**.

# ✅ Steps to Create a Custom Scope

---

## 1️. Implement `Scope` Interface

```java
import org.springframework.beans.factory.ObjectFactory;
import org.springframework.beans.factory.config.Scope;

import java.util.HashMap;
import java.util.Map;

public class ThreadScope implements Scope {

    private final ThreadLocal<Map<String, Object>> threadScope =
            ThreadLocal.withInitial(HashMap::new);

    @Override
    public Object get(String name, ObjectFactory<?> objectFactory) {
        Map<String, Object> scope = threadScope.get();
        return scope.computeIfAbsent(name, k -> objectFactory.getObject());
    }

    @Override
    public Object remove(String name) {
        return threadScope.get().remove(name);
    }

    @Override
    public void registerDestructionCallback(String name, Runnable callback) {
        // Optional (usually ignored for custom simple scopes)
    }

    @Override
    public Object resolveContextualObject(String key) {
        return null;
    }

    @Override
    public String getConversationId() {
        return Thread.currentThread().getName();
    }
}
```

---

## 2️. Register the Custom Scope

```java
import org.springframework.beans.factory.config.CustomScopeConfigurer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class ScopeConfig {

    @Bean
    public static CustomScopeConfigurer customScopeConfigurer() {
        CustomScopeConfigurer configurer = new CustomScopeConfigurer();
        configurer.addScope("thread", new ThreadScope());
        return configurer;
    }
}
```

---

## 3️⃣ Use the Custom Scope

```java
import org.springframework.context.annotation.Scope;
import org.springframework.stereotype.Component;

@Component
@Scope("thread")
public class MyThreadScopedBean {
}
```

Now:
- Same thread → same bean instance
- Different thread → different instance
---
# 🧠 When to Use Custom Scope?

- Multi-tenant applications
- Per-request logic outside web context
- Per-job (batch processing)
- Thread-bound resources
