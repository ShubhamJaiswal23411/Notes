## 1. Definition
The **N+1 problem** occurs when
- 1 query fetches a list of parent entities
- Then N additional queries are executed to fetch related entities (usually lazily)

So total queries = **1 + N**

---
##  Example Scenario

```java
@Entity
class Author {
    @Id
    Long id;

    @OneToMany(mappedBy = "author", fetch = FetchType.LAZY)
    List<Book> books;
}

@Entity
class Book {
    @Id
    Long id;

    @ManyToOne
    Author author;
}
```

Now:
```java
List<Author> authors = authorRepository.findAll();

for (Author author : authors) {
    System.out.println(author.getBooks().size());
}
```

---
##  What Actually Happens

1. Query 1:
```sql
SELECT * FROM author;
```

2. For each author ,let say n authors then n queries like this:
```sql
SELECT * FROM book WHERE author_id = ?;
```

If you fetched 100 authors → **101 queries**

---
##  Why This Is Dangerous
- Looks fine in dev (small data)
- Explodes in production
- Causes:
    - DB overload
    - Slow API responses
    - Hidden scalability bottleneck

---
##  Why It Happens
Because of:
```java
fetch = FetchType.LAZY
```

Hibernate uses **proxy objects** and only loads the relationship when accessed.

---
## 2. JPA Solutions 

### 1️. Use JOIN FETCH
```java
@Query("SELECT a FROM Author a JOIN FETCH a.books")
List<Author> findAllWithBooks();
```

Now:
```sql
SELECT a.*, b.* 
FROM author a 
JOIN book b ON b.author_id = a.id;
```
One query only.

---
### 2️. Use `@EntityGraph`

```java
@EntityGraph(attributePaths = {"books"})
List<Author> findAll();
```

Cleaner than JOIN FETCH for larger graphs.

---
## 3. Hibernate Solutions 

Hibernate gives more low-level control than pure JPA.
We’ll cover:
1. HQL (Hibernate Query Language)
2. FetchMode with Criteria API (old Hibernate API)
3. Batch fetching
4. Subselect fetching
5. Fetch profiles

---
### 1️. HQL (Hibernate Query Language)

HQL is Hibernate’s version of JPQL.
Using JOIN FETCH in HQL
```java
List<Author> authors = session.createQuery(
    "select distinct a from Author a join fetch a.books",
    Author.class
).getResultList();
```

Why `distinct`?

Because of row duplication caused by joins (Cartesian multiplication at SQL level).  
Hibernate deduplicates entities in memory.

---
### ⚠ Important

If you fetch multiple collections:

```java
join fetch a.books
join fetch a.awards
```

You may get:
- [[Cartesian explosion]]
- Or MultipleBagFetchException (if both are List)

---
### 2️. Hibernate Criteria API (Legacy API 'Depracated now')

You asked about this:
```java
Criteria criteria = session.createCriteria(Author.class);
criteria.setFetchMode("books", FetchMode.JOIN);
List<Author> authors = criteria.list();
```

```java
FetchMode.JOIN
```
It forces a SQL JOIN.

Fetch options are:

```java
FetchMode.JOIN
FetchMode.SELECT
FetchMode.SUBSELECT
```

What Each Does
1️. FetchMode.JOIN
	- Executes JOIN
	- Solves N+1
	- Risk: Cartesian explosion

2️. FetchMode.SELECT (default)
	- Separate SELECT per parent
	- Causes N+1

3️. FetchMode.SUBSELECT (Very Interesting)

```java
criteria.setFetchMode("books", FetchMode.SUBSELECT);
```

What happens:
1. Query authors:
```sql
select * from author;
```
2. Hibernate collects all author IDs
3. Executes ONE query:
```sql
select * from book 
where author_id in (list_of_all_loaded_authors);
```
Total queries: 2  
Not N+1  
Very powerful

---
### 3️. Hibernate Batch Fetching (Very Powerful)

Instead of fixing at query level, you let Hibernate optimize lazy loading.

### Option A — Global Config
```properties
hibernate.default_batch_fetch_size=16
```

### Option B — Per Association
```java
@BatchSize(size = 16)
@OneToMany(mappedBy = "author")
List<Book> books;
```

---
### What Happens Internally?

Instead of:
```sql
select * from book where author_id = 1;
select * from book where author_id = 2;
select * from book where author_id = 3;
```

Hibernate does:
```sql
select * from book 
where author_id in (1,2,3,...16);
```

Massive improvement.

---
### 4️. Subselect Fetching (Hibernate Specific)
Already shown in Criteria, but also works via annotation:
```java
@Fetch(FetchMode.SUBSELECT)
@OneToMany(mappedBy = "author")
List<Book> books;
```

This is a Hibernate-specific annotation:
```java
import org.hibernate.annotations.Fetch;
import org.hibernate.annotations.FetchMode;
```

### Important: Old Hibernate Criteria Is Deprecated

This:
```java
session.createCriteria()
```
Is deprecated in Hibernate 5+.

Modern approach:
- JPA CriteriaBuilder
- HQL
- Or QueryDSL

---
### When Should You Use What?

| Situation               | Best Hibernate Tool       |
| ----------------------- | ------------------------- |
| Always need association | JOIN FETCH (HQL)          |
| Large parent result set | SUBSELECT                 |
| Random lazy access      | Batch fetching            |
| Dynamic runtime control | Fetch Profiles            |
| Multiple collections    | Batch + secondary queries |

---

## 4. Senior-Level Insight

JOIN is not always best.
Sometimes:
- 1 big JOIN → 10,000 duplicated rows
- vs
- 2 optimized queries using SUBSELECT

Second approach is faster and uses less memory.

---
## 5. Important Insight

Even `EAGER` does NOT guarantee no N+1.
Hibernate may still generate multiple queries depending on the context.

---

## 6. Tricky Interview Questions (Very Important)

---
### 1️. If a relationship is EAGER, can N+1 still happen?
Yes.  
Because JPA spec says it _must be eagerly fetched_, but does NOT guarantee a single query.
### 2️. Why does `findAll()` often cause N+1 but `findById()` doesn’t?
Because:
- `findById()` loads one parent
- Lazy loading triggers once
- Not N times
### 3️. Can N+1 happen with ManyToOne?
Yes.  
If you fetch many children and access parent lazily.
### 4️. Why doesn’t Hibernate automatically solve N+1?
Because:
- It cannot predict which associations you'll access
- Fetching everything eagerly would cause Cartesian explosion
### 5️. How does `@EntityGraph` differ from JOIN FETCH?
`@EntityGraph`:
- Declarative
- Cleaner
- Can be reused

JOIN FETCH:
- Explicit JPQL
- More control

# 🏁 Final Mental Model

| Concept                 | Safe?     | Risk                     |
| ----------------------- | --------- | ------------------------ |
| LAZY                    | Yes       | N+1                      |
| EAGER                   | Sometimes | Cartesian explosion      |
| JOIN FETCH              | Yes       | Over-fetching            |


### 6. Does FetchMode.JOIN always solve N+1?

> It prevents N+1 but may introduce Cartesian explosion and duplicate rows. It’s not always the optimal solution.

### 7. Why is SUBSELECT sometimes better than JOIN?
Because:
- It avoids row multiplication
- Keeps result sets smaller
- Still avoids N+1
