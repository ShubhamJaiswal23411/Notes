
##  What `@Data` Generates

```java
@Data
```

Generates:
- getters
- setters
- `toString()`
- `equals()`
- `hashCode()`
- `requiredArgsConstructor`

---
## 1. Why This Is Bad for JPA Entities

JPA entities are NOT simple POJOs.
They:
- Have lazy proxies
- Have bidirectional relationships
- Rely on identity semantics
- Can exist in detached states

`@Data` breaks multiple assumptions.

---

# 2. Problem 1 — Lazy Loading Triggered by `toString()`
If your entity has:
```java
@OneToMany(mappedBy = "author")
List<Book> books;
```

Lombok generates:
```java
public String toString() {
    return "Author(books=" + this.getBooks() + ")";
}
```

If this runs:
- Logging
- Debugging
- Printing entity

It triggers lazy loading → **N+1 explosion**

---
# 3. Problem 2 — Infinite Recursion

Bidirectional relationship:
```java
Author -> books
Book -> author
```

Generated `toString()`:
- Author prints books
- Book prints author
- Author prints books again

we get a StackOverflowError

---

# 4. Problem 3 — Broken equals() / hashCode()

Lombok includes ALL fields by default.
That means:
```java
equals() compares:
- id
- books
- relationships
```

Problems:
###  1. Performance
Comparing collections triggers lazy loading.

###  2. Hibernate Identity Violation
Entities should usually compare by:
- Database primary key  
    OR
- Business key
Not by collections.

###  3. Transient Entity Problem
Before persisting:
```java
id == null
```

After persisting:
```java
id != null
```

If equals/hashCode uses id:
- HashSet behavior breaks
- Entity disappears from sets

---
# 5. Problem 4 — Foreign Keys & Proxy Issues

When entity has:
```java
@ManyToOne
@JoinColumn(name = "author_id")
Author author;
```

Hibernate may store a proxy:
```
Author$HibernateProxy
```

If equals/hashCode uses the entire object:
- Proxy != actual entity
- Equals fails
- Weird collection bugs happen

---
# 6. Correct Way to Write JPA Entities

###  DO NOT use `@Data`
Instead:
```java
@Getter
@Setter
@NoArgsConstructor
```

Manually define equals/hashCode carefully.

---

##  Safe equals/hashCode Pattern

Best practice (ID-based, but careful):

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (!(o instanceof Author)) return false;
    return id != null && id.equals(((Author) o).id);
}

@Override
public int hashCode() {
    return getClass().hashCode();
}
```

This avoids:
- Lazy loading
- Proxy comparison issues
- HashSet corruption



#  7. Lombok + JPA Interview Questions

---
### 1. Why is `@Data` dangerous for JPA entities?
Because it generates:
- equals/hashCode including relationships
- toString triggering lazy loading
- breaks Hibernate identity semantics

---
### 2. What happens if equals() uses a collection field?
- Triggers lazy loading
- Can cause N+1
- Can cause recursion

---
### 3. Why should hashCode NOT use ID directly?
Because:
- ID changes after persist
- Breaks hash-based collections

---

### 4. What problem occurs when comparing Hibernate proxies?
Proxy class != actual entity class  
equals() may return false even if IDs match.

---
### 5. What is the safest Lombok usage with JPA?
```java
@Getter
@Setter
@NoArgsConstructor
```

Never:
```java
@Data
```
