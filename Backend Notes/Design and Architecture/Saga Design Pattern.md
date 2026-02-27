
The **Saga pattern** is a microservices pattern used to manage **distributed transactions** across multiple services **without using 2-Phase Commit (2PC)**.

In microservices, each service:
- Has its own database
- Is independently deployable
- Cannot participate easily in a global transaction

A **Saga** ensures **data consistency** across services by breaking a large transaction into **a sequence of local transactions**.

Each step:
1. Performs a local transaction
2. Publishes an event (or calls next service)
3. If something fails → triggers **compensating transactions** (rollback steps)

---

# 🧩 Example Scenario (Order Service)

Suppose we have:

- Order Service
- Payment Service
- Inventory Service
- Shipping Service

Flow:

1. Create Order
2. Reserve Inventory
3. Process Payment
4. Ship Order

If payment fails →  
Inventory reservation must be canceled →  
Order must be marked as failed.

This coordinated rollback = **Saga compensation**

---

# 📌 How Many Types of Saga?

There are **2 types**:

---

# 1️. Choreography-Based Saga

### 👉 No central coordinator

Services communicate through events.

Each service:
- Listens to events
- Performs its transaction
- Publishes the next event

### Example Flow
1. OrderCreated → Inventory reserves
2. InventoryReserved → Payment processes
3. PaymentFailed → Inventory releases
4. InventoryReleased → Order cancelled

### Characteristics
- Event-driven
- Decentralized
- Uses message broker (Kafka, RabbitMQ)

---

# 2️. Orchestration-Based Saga

### 👉 Central orchestrator controls flow

A separate **Saga Orchestrator**:
- Calls each service
- Decides next step
- Triggers compensations if failure happens

### Example Flow
1. Orchestrator → Create Order
2. Orchestrator → Reserve Inventory
3. Orchestrator → Process Payment
4. If failure → Orchestrator triggers compensations

### Characteristics
- Centralized control
- Clear flow visibility
- Easier to monitor

---

# 🆚 Choreography vs Orchestration

| Feature    | Choreography        | Orchestration           |
| ---------- | ------------------- | ----------------------- |
| Control    | Distributed         | Central                 |
| Coupling   | Loosely coupled     | More controlled         |
| Complexity | Grows with services | Easier to manage        |
| Visibility | Harder to trace     | Easier to track         |
| Risk       | Event chaos         | Single point of control |

---

# ✅ Advantages of Saga Pattern
1. No need for 2PC (better scalability)
2. Maintains eventual consistency
3. Services remain autonomous
4. Fault tolerant with compensation
5. Works well with event-driven systems

---

# ❌ Disadvantages
1. Complex error handling
2. Compensating logic must be carefully written
3. Debugging distributed flow is hard
4. Event ordering issues
5. Hard to guarantee strong consistency
6. Orchestrator can become bottleneck (in orchestration)

---

# How to Implement Saga in Microservices

## 1️. Using Choreography (Event-Driven)

Tools:
- Kafka
- RabbitMQ
- EventBridge

Each service:
- Listens to events
- Publishes next event
- Implements compensation logic

Spring Boot Example Concept:

```java
@KafkaListener(topics = "order-created")
public void reserveInventory(OrderCreatedEvent event) {
    // reserve inventory
    // publish InventoryReservedEvent
}
```

---

## 2️. Using Orchestration

Use a Saga orchestrator:
Options:
- Custom orchestrator service
- Workflow engines:
    - Temporal
    - Camunda
    - Netflix Conductor
    - Axon Framework

Spring Boot simple orchestrator example:
```java
public void processOrder(Order order) {
    try {
        orderService.create(order);
        inventoryService.reserve(order);
        paymentService.process(order);
    } catch (Exception e) {
        compensate(order);
    }
}
```

In real systems → usually asynchronous + state machine.

---

# 🔥 When Should You Use Saga?

Use Saga when:
- You have distributed services with separate databases
- You need consistency across services
- 2PC is not feasible
- High scalability is required

Avoid Saga when:
- Strong ACID consistency is mandatory
- System is small and monolithic
- Business logic is simple

---

# ✅ What is 2-Phase Commit (2PC)?

**2-Phase Commit (2PC)** is a distributed transaction protocol that ensures **atomicity** across multiple services/databases.

It uses a **transaction coordinator** and happens in two phases:

---

## 🔹 Phase 1: Prepare Phase

1. Coordinator asks all participants: _“Can you commit?”_
2. Each service:
    - Executes transaction
    - Locks required resources
    - Responds YES or NO

---

## 🔹 Phase 2: Commit/Rollback Phase
- If **all say YES** → Coordinator sends COMMIT
- If **any says NO** → Coordinator sends ROLLBACK

All services then either commit or rollback.

---

## ⚠️ Problems with 2PC in Microservices
- Services must hold **locks**
- Blocking protocol
- Poor scalability
- Not cloud-native friendly
- Tight coupling between services
- Coordinator is a single point of failure

That’s why Saga is preferred in microservices.

---

# ✅ Saga vs 2PC Comparison

| Feature               | 2PC                       | Saga                             |
| --------------------- | ------------------------- | -------------------------------- |
| Consistency           | Strong consistency (ACID) | Eventual consistency             |
| Locking               | Holds locks until commit  | No long-held locks               |
| Scalability           | Poor                      | High                             |
| Coupling              | Tight                     | Loosely coupled                  |
| Failure Handling      | Global rollback           | Compensating transactions        |
| Performance           | Slower (blocking)         | Faster (async)                   |
| Cloud-native friendly | ❌ Not ideal               | ✅ Designed for it                |
| Complexity            | Protocol complexity       | Business compensation complexity |

---

## 🎯 Core Difference

- **2PC = atomic distributed transaction**
    
- **Saga = sequence of local transactions with compensations**
    

2PC guarantees strict consistency.  
Saga trades strict consistency for scalability and resilience.
