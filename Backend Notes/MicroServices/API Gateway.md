## What is an API Gateway?

An **API Gateway** is a server that acts as a **single entry point** for all client requests in a microservices architecture.

Instead of this:
```text
Client → user-service
Client → order-service
Client → payment-service
```

You have this:
```text
Client → API Gateway → microservices
```

The gateway sits between clients and backend services.

---

## What Does an API Gateway Do?

An API Gateway typically handles:
- Routing
- Authentication & Authorization
- Rate limiting
- Load balancing
- SSL termination
- Logging & monitoring
- Request/Response transformation
- Aggregation of multiple services

---

## Examples of API Gateways

- Spring Cloud Gateway
- Kong
- NGINX
- AWS API Gateway

---

# Why Do We Need an API Gateway?

## Problem Without Gateway

If you expose all microservices directly:

- Client must know all service URLs
- Authentication logic duplicated
- Complex frontend logic
- Tight coupling
- Security risks

Example:

```text
Mobile app must call:
- /users
- /orders
- /payments
- /inventory
```

Each service must handle:

- Auth
- Rate limits
- Logging

That’s messy.

---

## With API Gateway

```text
Mobile App → API Gateway → Services
```

Now:
- Only one public endpoint
- Centralized authentication
- Centralized logging
- Controlled traffic
- Easier versioning

---

# Practical Application Example

## 🛒 E-commerce System

Microservices:
- user-service
- order-service
- payment-service
- inventory-service

### Scenario: User Views Order Details

Frontend needs:

- User info
- Order info
- Payment status
- Product details

Without gateway:
- 4 HTTP calls from frontend

With gateway:
- 1 call to gateway
- Gateway aggregates responses
- Returns combined response

This is called **API Aggregation Pattern**.

---

# API Gateway + Service Discovery

API Gateway works together with:

- Netflix Eureka
- Kubernetes

Flow:

```text
Client → Gateway → Registry/DNS → Service Instance
```

The gateway:
- Discovers available instances
- Chooses one
- Forwards request

---

# API Gateway vs Load Balancer

|Feature|Load Balancer|API Gateway|
|---|---|---|
|Distributes traffic|✅|✅|
|Authentication|❌|✅|
|Rate limiting|❌|✅|
|Aggregation|❌|✅|
|Protocol transformation|❌|✅|

Load balancer = traffic distributor  
API Gateway = smart traffic manager



# Real-World Practical Uses

---

## 1️. Security Layer

Gateway validates:
- JWT tokens
- OAuth tokens
- API keys

Microservices do NOT need to implement auth repeatedly.

---

## 2️. Rate Limiting

Prevent abuse:
```text
Max 100 requests per minute per user
```

Handled centrally.

---

## 3️. API Versioning

```text
/api/v1/users
/api/v2/users
```

Gateway routes versions correctly.

---

## 4️. Monitoring & Logging

Centralized:
- Request logs
- Response times
- Error tracking

---

## 5️. Circuit Breaking & Resilience

If payment-service fails:
- Gateway can return fallback response
- Prevents system-wide failure

---

# When Should You Use API Gateway?

Use it when:
- You have multiple microservices
- You expose APIs publicly
- You need centralized security
- You want clean frontend architecture
- You deploy in cloud/Kubernetes

Avoid it when:
- Monolithic application
- Very small internal system

---

# Production-Level Architecture Example

## Modern Cloud Setup

```text
Internet
   ↓
API Gateway
   ↓
Kubernetes Service
   ↓
Pods (multiple instances)
```

Gateway handles:
- Auth
- Rate limiting
- Routing

Kubernetes handles:
- Scaling
- Pod networking
- Internal load balancing

---

# Interview-Level Explanation

> An API Gateway is a centralized entry point that routes client requests to appropriate backend services while handling cross-cutting concerns such as authentication, rate limiting, logging, and request aggregation. It simplifies client interactions and improves security and scalability in microservices architectures.


# 1️. If API Gateway is doing load balancing, how is Kubernetes also doing it?

👉 Short answer: **They load balance at different layers.**

---

## 🔹 Layer 1: API Gateway Load Balancing (North–South Traffic)

This is for **external traffic** (outside → inside cluster).

Example:

```text
Internet → API Gateway → user-service
```

If `user-service` has 3 pods:

```text
user-service
  ├─ pod-1
  ├─ pod-2
  └─ pod-3
```

Gateway can:
- Pick one pod directly (if integrated with discovery)
- OR forward to Kubernetes Service (which then load balances)
    

---

## 🔹 Layer 2: Kubernetes Load Balancing (East–West Traffic)

In Kubernetes, every **Service**:
- Has a stable virtual IP
- Has a DNS name
- Automatically distributes traffic across pods

Example:
```text
order-service → http://user-service
```

Kubernetes internally load balances between pods using:
- kube-proxy
- iptables/IPVS

---

### 🧠 So Who Actually Does It?

Common production setup:

```text
Client
   ↓
API Gateway
   ↓
Kubernetes Service
   ↓
Pods
```

So:
- Gateway handles routing + security
- Kubernetes handles pod-level balancing

They are not conflicting — they operate at different levels.

---

# 2️. Should Microservices Call Each Other Through API Gateway?

👉 Best Practice: **NO.**

Internal microservice-to-microservice communication should be direct.
## 🔹 External Traffic (North–South)

```text
Client → API Gateway → Services
```

Use Gateway ✅

---

## 🔹 Internal Traffic (East–West)

```text
order-service → user-service
```

Direct call via:

```text
http://user-service
```

If using Kubernetes:
- DNS resolves service
- Kubernetes load balances

You do NOT need API Gateway for internal calls.
### Why Not Use Gateway Internally?
- Extra network hop
- Increased latency
- Gateway becomes bottleneck
- Tighter coupling

Gateway is for:
- Security boundary
- External exposure

---

# 3️. How Does API Gateway Do These Things?

Let’s go one by one.
## 🔐 A) Validation of Auth Tokens

Example with JWT:
Client sends:
```text
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

Gateway:
1. Extracts token
2. Verifies signature
3. Checks expiration
4. Validates roles/claims

If invalid:
```text
401 Unauthorized
```

If valid:
- Forward request
- Optionally attach user info in headers

Example technologies:
- Spring Cloud Gateway
- Kong
- AWS API Gateway

This prevents each microservice from re-implementing authentication.

---

## 📦 B) Aggregation (What It Means)

Aggregation = **Combining multiple backend responses into one single response.**

---

### Without Aggregation

Frontend makes 3 calls:
```text
GET /users/1
GET /orders/1
GET /payments/1
```

---

### With Aggregation
Frontend makes 1 call:
```text
GET /user-dashboard/1
```

Gateway internally:
```text
Calls user-service
Calls order-service
Calls payment-service
Combines responses
Returns single JSON
```

Example combined response:
```json
{
  "user": {...},
  "orders": [...],
  "payments": {...}
}
```

This:
- Reduces frontend complexity
- Reduces network calls
- Improves performance

---

## 🔄 C) Protocol Transformation

API Gateway can translate between protocols.
Example scenarios:
### 1️. HTTP → gRPC

Client sends HTTP/JSON  
Backend service uses gRPC

Gateway:
- Converts JSON → Protobuf
- Makes gRPC call
- Converts response back to JSON

---

### 2️. HTTP → WebSocket

Gateway upgrades connection for real-time apps.

---

### 3️. HTTP → Internal Service Mesh

Gateway hides internal communication protocol.

---

# 🔎 Summary of Responsibilities

|Feature|API Gateway|Kubernetes|
|---|---|---|
|External routing|✅|❌|
|Pod load balancing|⚠️ Sometimes|✅|
|Auth validation|✅|❌|
|Aggregation|✅|❌|
|Protocol transform|✅|❌|
|Internal service discovery|❌|✅|

---

# 🔥 Final Architecture View

In modern systems:

```text
Internet
   ↓
API Gateway (security + routing)
   ↓
Kubernetes Service (discovery + load balancing)
   ↓
Pods
```

Internal services:

```text
service-A → service-B (direct via K8s DNS)
```

