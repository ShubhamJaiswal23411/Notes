
In distributed systems:
- Messages can be duplicated
- Services can retry
- Network failures can occur
So each step must be **idempotent**.

---

##  1. What is Idempotency?

An operation is idempotent if:
> Running it multiple times produces the same result as running it once.

Example:
Bad:
- Charge credit card twice → double payment ❌
Good:
- If payment already processed → ignore duplicate request ✅

---

##  2. How to Achieve Idempotency

### 1️. Use Unique Transaction IDs

Every saga step has a unique ID.
Before processing:
- Check if ID already processed
- If yes → ignore

Example:
```sql
CREATE TABLE processed_events (
    event_id VARCHAR PRIMARY KEY
);
```

---

### 2️. Use Idempotency Keys (For APIs)

Client sends:
```
Idempotency-Key: 12345
```

Service stores result for that key.
Repeated calls return same response.

---

### 3️. Database Constraints

Use:
- Unique constraints
- Upserts
- Version checks

---

### 4️. Status-Based Processing

Instead of:
```
chargePayment()
```

Use:
```
if paymentStatus != SUCCESS → process
```

---

#  2. Retry Strategies in Saga

Failures are common in distributed systems.
Retry strategies help recover from temporary issues.

---

## 🔹 1️. Automatic Retries (Transient Failures)

Retry when:
- Network timeout
- Temporary DB issue
- Service unavailable

Use:
- Exponential backoff
- Limited retry attempts

Example:
Retry after:
- 1 sec
- 2 sec
- 4 sec
- 8 sec

---

## 🔹 2️. Dead Letter Queue (DLQ)

If retries fail:
- Send event to DLQ
- Manual investigation
- Avoid infinite retry loops

---

## 🔹 3️. Compensation After Retry Failure

If step fails permanently:
- Trigger compensating transactions

Example:
- Payment fails → release inventory → cancel order

---

## 🔹 4️. Timeout Handling

If a service does not respond:
- Mark step as failed
- Trigger compensation
- Or retry

---

# 🔥 Important: Idempotency + Retry Together

Retries without idempotency = dangerous.

If you retry payment without idempotency:  
→ Customer may be charged multiple times.

So rule:
> Every retriable operation in Saga must be idempotent.

---

# 🧠 Real-World Pattern

In production systems:
- Use message broker (Kafka, RabbitMQ)
- Store Saga state in DB
- Use unique event IDs
- Use retry with backoff
- Use DLQ
- Make every step idempotent
- Monitor saga state transitions


**Idempotency** prevents duplicate effects during retries.
**Retries** handle transient failures using backoff and DLQ, but must always be combined with idempotent operations.
