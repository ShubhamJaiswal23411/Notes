 # 1️. What Is a Transaction?

A **transaction** is a logical unit of work that must satisfy **ACID** properties:
- **A**tomicity → All or nothing
- **C**onsistency → DB moves from valid state to valid state
- **I**solation → Transactions don’t see inconsistent data
- **D**urability → Once committed, data survives crashes

In Spring Boot, transaction management is built on:
- `@Transactional`
- AOP proxies
- PlatformTransactionManager
- Usually Hibernate + JDBC underneath

---
#  2️. How Spring Transaction Management Works Internally

When you add:
```java
@Transactional
public void transferMoney() {}
```

Spring:
1. Creates a proxy around the bean
2. Proxy intercepts method call
3. Starts transaction before method
4. Commits or rolls back after method
5. Releases DB connection

-  Important:  
- Transactions are applied via **proxy-based AOP**

That leads to one of the most famous interview traps (we’ll cover later).

---
#  3️. Transaction Scope

Transaction scope = The boundary of execution where the transaction is active.
Default scope:
- Method level
- Thread bound (ThreadLocal)
- Bound to DB connection

---
# 4️. Propagation Types (VERY IMPORTANT)

Propagation defines:

> What happens if a transactional method is called inside another transactional method?

## 1. REQUIRED (Default)

```java
@Transactional(propagation = Propagation.REQUIRED)
```

Behavior:
- If transaction exists → join it
- If not → create new one
-  Most common  
- Inner rollback affects outer

---
## 2. REQUIRES_NEW

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
```

Behavior:
- Suspend existing transaction
- Create completely new transaction
- Independent commit/rollback  
- Outer transaction unaffected by inner rollback

---
## 3. NESTED

```java
@Transactional(propagation = Propagation.NESTED)
```

Behavior:
- Creates savepoint inside existing transaction
- Can rollback to savepoint
- Outer transaction continues
-  ***Important point : Only works with JDBC + DataSourceTransactionManager***  
- ***Not fully supported with JPA + some drivers***

---
## 4. SUPPORTS

- If transaction exists → join
- Else → run without transaction

---
## 5. NOT_SUPPORTED
- Suspend existing transaction
- Execute non-transactionally

---
## 6. MANDATORY

- Must run inside transaction
- Else → Exception

---
## 7. NEVER
- Must NOT run inside transaction
- Else → Exception

---

# 5️. Isolation Levels

Controls visibility of uncommitted data.

|Isolation|Prevents|
|---|---|
|READ_UNCOMMITTED|Nothing|
|READ_COMMITTED|Dirty reads|
|REPEATABLE_READ|Dirty + Non-repeatable reads|
|SERIALIZABLE|Dirty + Non-repeatable + Phantom|

Spring:

```java
@Transactional(isolation = Isolation.REPEATABLE_READ)
```

Actual behavior depends on DB (e.g., MySQL default = REPEATABLE_READ).

---

#  6️. Which Exceptions Trigger Rollback?

## Default Rule:
Spring rolls back ONLY on:
- RuntimeException
- Error

It DOES NOT rollback on:
- Checked exceptions (Exception)

---
## Example

```java
@Transactional
public void test() throws Exception {
    throw new Exception(); // Checked
}
```

Transaction will COMMIT ❗

---

#  7️. How To Rollback For Checked Exception

## Option 1 — rollbackFor

```java
@Transactional(rollbackFor = Exception.class)
```

## Option 2 — Custom Exception

```java
@Transactional(rollbackFor = MyCheckedException.class)
```

---
## Option 3 — Programmatic Rollback

```java
TransactionAspectSupport.currentTransactionStatus()
    .setRollbackOnly();
```

---

#  8️. The Most Important Propagation Scenarios

Now the tricky part.

## Scenario 1: REQUIRED Inside REQUIRED

```java
@Transactional
public void outer() {
    inner();
}

@Transactional
public void inner() {
    throw new RuntimeException();
}
```

Result:
- Single transaction
- Inner throws RuntimeException
- Entire transaction rolls back
- Outer also rolled back
✔ Shared transaction

---
# Scenario 2: REQUIRES_NEW Inside REQUIRED

```java
@Transactional
public void outer() {
    inner();
}

@Transactional(propagation = REQUIRES_NEW)
public void inner() {
    throw new RuntimeException();
}
```

Result:
- Outer transaction suspended
- Inner transaction starts
- Inner rolls back
- Outer continues normally

✔ Inner rollback does NOT affect outer

---
# Scenario 3: REQUIRED Inside REQUIRES_NEW

Outer:
```java
@Transactional(REQUIRES_NEW)
```

Inner:
```java
@Transactional(REQUIRED)
```

Result:
- Inner joins outer transaction
- If inner fails → whole thing rolls back

---
# Scenario 4: NESTED Inside REQUIRED

```java
@Transactional
public void outer() {
    try {
        inner();
    } catch (Exception ignored) {}
}
```

```java
@Transactional(propagation = NESTED)
public void inner() {
    throw new RuntimeException();
}
```

Result:
- Savepoint created
- Inner rolls back to savepoint
- Outer continues
- Outer can still commit
✔ Partial rollback

---

#  9️. Tricky Interview Questions

##  1. Why doesn’t @Transactional work on private methods?
Because Spring uses proxies.
Internal method calls bypass proxy → no transaction created.

##  2. What happens if a transactional method calls another method in the same class?
Transaction annotation on inner method is ignored.
Self-invocation problem.

##  3. If inner transaction (REQUIRES_NEW) commits but outer rolls back, what happens?
Inner stays committed.
Because it was independent.

##  4. If outer commits but inner REQUIRES_NEW rolls back?
Outer still commits.

##  5. Can NESTED work without an outer transaction?
No.  
It behaves like REQUIRED.

##  6. What happens if inner REQUIRED throws checked exception?
By default:  
No rollback.
Unless configured.

##  7. Can transaction span multiple threads?
No.  
Transactions are thread-bound (ThreadLocal).

##  8. Does @Transactional work on @Async methods?
No.  
Different thread → no transaction context.

##  9. What happens if you catch exception inside transaction?
```java
@Transactional
public void test() {
    try {
        risky();
    } catch (Exception e) {}
}
```

Transaction commits.
Because exception never propagated.

##  10. What happens if you mark rollbackOnly but method completes normally?

Transaction rolls back at commit time.

---

#  10️. Most Dangerous Real-World Bugs

-  Self-invocation
- Checked exception not rolling back
- Using REQUIRES_NEW inside loops (creates too many transactions)
- Long-running transactions blocking DB
- Mixing @Transactional with async

---

#  Senior-Level Mental Model

Transactions in Spring are:
- Proxy-based
- Thread-bound
- Exception-driven
- Connection-scoped
- Declarative abstraction over JDBC

Understanding propagation = understanding how transactions suspend/resume.

---

#  Final Interview Killer Question

> If outer REQUIRED transaction calls inner REQUIRES_NEW, inner commits, outer rolls back — what is final DB state?

Answer:
- Inner changes are committed
- Outer changes are rolled back
- Because they were separate physical transactions

---

# Golden Rule

Propagation determines:
> Whether transactions share the same physical connection or suspend it.

Rollback depends on:
> Exception type + propagation + catching behavior.
