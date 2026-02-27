
## 1️. Service Registry

### Definition

A **Service Registry** is a **central database/server** where services:

- Register themselves
- Send heartbeats
- Store metadata (IP, port, health status)
- Deregister on shutdown

It is basically the **directory of services**.

### What it Stores

- Service name
- Instance ID
- IP address
- Port
- Health status
- Metadata

### Examples

- Netflix Eureka
- Consul
- Apache ZooKeeper

---

## 2️. Service Discovery

### Definition

**Service Discovery** is the **process/mechanism** by which a service:

- Finds another service
- Queries the registry
- Gets available instances
- Selects one (often via load balancing)

👉 Registry = storage  
👉 Discovery = lookup + selection process

---

## 3️. How They Work Together

### Flow

1. User Service starts
2. Registers with registry
3. Order Service wants to call User Service
4. Order Service queries registry
5. Gets list of instances
6. Chooses one (load balancing)
7. Sends request

---

## 4️. Do They Come With Load Balancer by Default?

### Short Answer: ❌ Not always

It depends on the ecosystem.

---

### Case 1: Using Spring Cloud Netflix with Netflix Eureka
- Eureka → only registry
- Load balancing handled by:
    - Spring Cloud LoadBalancer (modern)
    - Ribbon (older, deprecated)

So:
> Registry ❌ does NOT automatically mean load balancer

You need:
```id="0y3z9x"
@LoadBalanced RestTemplate
```

---

### Case 2: Kubernetes
Kubernetes has:
- Built-in service registry (via DNS)
- Built-in load balancing (via Services)

So in Kubernetes:
> Service + kube-proxy → automatic load balancing

---

### Case 3: Consul
Consul:
- Registry 
- Optional load balancing (via service mesh like Envoy)

---

## 5️. Scaling Scenario (Pods & Registration)

### Scenario: Spring Boot in Kubernetes
If a deployment scales:
```id="az7pm1"
user-service → 1 pod → 3 pods
```

Each pod:
- Has its own IP
- Has its own container port
- Registers itself

So yes:

>  Each new pod registers its IP and port

But how this happens depends on environment.

---

### In Eureka-based System

When new instance starts:

```id="wafl3m"
user-service-1 → registers
user-service-2 → registers
user-service-3 → registers
```

Registry now stores 3 entries.
Clients discover all 3 and load balance between them.

---
### In Kubernetes
Pods DO NOT manually register in Eureka (usually).
Instead:
- Pods are part of a Service
- Kubernetes DNS automatically maps:

```id="m0r1qp"
user-service.default.svc.cluster.local
```

- Kube-proxy distributes traffic across pods

Here:
- No manual registration
- Kubernetes manages endpoints automatically

---

## 6️. Important Difference in Cloud-Native World

### Traditional Microservices (VM-based)
- Explicit service registry (Eureka)
- Services self-register
### Kubernetes-native
- Registry is replaced by:
    - Kubernetes API
    - DNS
    - Service abstraction
No separate registry server required.

