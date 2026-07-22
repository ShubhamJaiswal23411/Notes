
# What Is .setRollBackOnly()?

```java
TransactionAspectSupport.currentTransactionStatus()
    .setRollbackOnly();
```

This tells Spring:

> “Do NOT commit this transaction when the method finishes — roll it back instead.”

It does **not** immediately rollback.  
It just marks the transaction as **rollback-only**.

Actual rollback happens at **commit phase**.

---

#  Why Would You Use It?

Because sometimes:
- You **don’t want to throw an exception**
- But you **still want the transaction to rollback**

Spring by default only rolls back on:
- RuntimeException
- Error

If you:
- Catch exception
- Handle error manually
- Or have business validation failure

Transaction would normally commit.
`setRollbackOnly()` gives you manual control.

---
# When To Use It

## 1️. When You Catch an Exception But Still Want Rollback

```java
@Transactional
public void process() {
    try {
        riskyOperation();
    } catch (Exception e) {
        log.error("Error occurred", e);

        TransactionAspectSupport
            .currentTransactionStatus()
            .setRollbackOnly();
    }
}
```

Without `setRollbackOnly()`:  
- Exception caught  
- Method completes normally  
- Transaction commits

With it:  
- Transaction rolls back

---

## 2️. Business Rule Validation Failure (No Exception)

Sometimes failure is not “exceptional”.
Example:
```java
@Transactional
public void transfer(Account from, Account to, double amount) {
    if (from.getBalance() < amount) {
        TransactionAspectSupport
            .currentTransactionStatus()
            .setRollbackOnly();
        return;
    }

    from.debit(amount);
    to.credit(amount);
}
```

You don’t want to throw exception.  
But you also don’t want partial DB changes.

---
## 3️. Partial Flow Decision Late in Method

Sometimes error is detected **after multiple DB writes**.
Instead of throwing, you decide:
> “Undo everything.”

That’s when marking rollback-only makes sense.

---

#  Very Tricky Behavior (Interview Trap)

## What If Inner Method Marks RollbackOnly?

```java
@Transactional
public void outer() {
    inner();
}

@Transactional
public void inner() {
    TransactionAspectSupport.currentTransactionStatus()
        .setRollbackOnly();
}
```

Propagation = REQUIRED (default)
Result:
- Single transaction
- Inner marks rollbackOnly
- Outer continues normally
- At commit → Spring throws:

```text
UnexpectedRollbackException
```

Why?
Because outer thinks everything is fine,  
but transaction was silently marked rollback-only.

Reason from spring perspective :
Spring enforces this rule:
> If a method completes normally, the transaction should commit.

If it doesn’t commit, that is unexpected behavior
So the inner method only marks the setRollBack = true.
No exception was thrown from the outer method so transcation should commit but setRollBack was true to spring will rollback the tracsaction but it wil throw UnexpectedRollbackException.

***Spring prevents silent rollback.***


---

# 🧠 Difference Between Throwing Exception vs setRollbackOnly()

| Throw Exception                 | setRollbackOnly()   |
| ------------------------------- | ------------------- |
| Immediately exits method        | Method continues    |
| Cleaner design                  | Manual control      |
| Triggers rollback automatically | You decide rollback |
| Simpler                         | More complex        |

---

# 🎯 Why Not Just Throw Exception?

Sometimes:
- You want to return custom response
- You want to avoid propagating exception
- You’re inside large flow and don’t want stack unwinding
- You’re inside legacy system where exceptions are swallowed

That’s when this is useful.

---

#  Senior-Level Mental Model

Transactions are committed at method exit.

`setRollbackOnly()` is like telling Spring:

> “When you reach the finish line, abort instead of commit.”

It doesn’t cancel execution.  
It changes the final outcome.

---

#  Real Production Example

Imagine:
- 5 DB operations succeeded
- Final validation fails
- You want to log and return structured response
- Not throw exception to controller

You mark rollback-only and return:

```json
{
  "status": "FAILED",
  "reason": "Business rule violation"
}
```

But DB changes never persist.

---
# 🏁 Final Summary

Use:
```java
TransactionAspectSupport.currentTransactionStatus()
    .setRollbackOnly();
```

When:
- You want rollback
- But you do NOT want to throw exception
- Or you caught the exception already
- Or rollback depends on business logic

It marks the transaction for rollback at commit time.

