
##  Logs vs Distributed Tracing

###  Logs
Logs record events during request processing (text/JSON).

### Useful for:
- Debugging (what happened inside a service)
- Error tracking (stack traces)
- Monitoring behavior

###  Logs capture:
- Incoming request (URL, headers, body)
- Internal processing events
- Response (status, time, payload)

### Limitations:
- No end-to-end visibility across services
- Cannot measure time spent per service
- Hard to correlate logs across microservices

---
##  What is Distributed Tracing?

Distributed tracing tracks a request **across multiple microservices**.
###  It shows:
- Service-to-service calls
- Time taken by each service
- Status of each step
###  Important:
- It gives **high-level flow**, not detailed logs

---
##  Core Concepts

###  Trace ID
- Unique ID for the **entire request**
- Same across all services
###  Span ID
- Represents **one unit of work** (per service/action)

---
###  Example Flow

```
OrderService → PaymentService
```

Logs:

```
TRACE-ID=T123 [OrderService] Order created
TRACE-ID=T123 [PaymentService] Payment failed
```

 Using the same Trace ID, logs can be stitched in tools like Kibana/CloudWatch.

---
##  Why Tracing + Logging Together?
- Tracing → shows **flow + latency**
- Logs → show **detailed debugging info**

👉 Combined = full observability

---
##  Legacy vs Modern Approach

### Old (Spring Cloud Sleuth + Brave)
- Tight coupling with Zipkin
- Harder to integrate with other tools
###  Modern (Micrometer + OpenTelemetry)
- Vendor-neutral
- Works with Jaeger, Zipkin, etc.

---
##  Implementation (High-Level)

###  Step 1: Start Backend (Jaeger)

```bash
docker run -d -p 16686:16686 -p 4317:4317 -p 4318:4318 jaegertracing/all-in-one
```

- 6686 → Jaeger UI (open browser: https://localhost:16686)
- 4317 → OTLP gRPC endpoint 
- 4318 → OTLP HTTP endpoint

### Step 2: Add Dependencies

```xml
spring-boot-starter-actuator
micrometer-tracing-bridge-otel
opentelemetry-exporter-otlp
```

###  Step 3: Configuration

```properties
spring.application.name=app-name
management.tracing.sampling.probability=1.0
management.otlp.tracing.endpoint=http://localhost:4318/v1/traces
```

###  Step 4: Automatic Tracing
Out of the box, spans are created for:
- Incoming HTTP requests
- Outgoing HTTP calls
- Thread switching

---
##  Service-to-Service Tracing

###  Important: Context Propagation
Headers automatically added:
```
traceparent: <trace-id>-<span-id>
```

---
###  Using RestClient (Recommended)

```java
@Bean
RestClient restClient(RestClient.Builder builder) {
    return builder.build(); // auto propagation works
}
```

---
###  Common Mistake

```java
RestClient.create(); // ❌ No tracing propagation
```

👉 Because interceptors are not added

---
##  Manual Span Creation (Advanced)

```java
Span parent = tracer.currentSpan();

Span child = tracer.nextSpan(parent).name("child-span").start();

try (Tracer.SpanInScope scope = tracer.withSpan(child)) {
    // business logic
} finally {
    child.end();
}
```

---
###  Result:
- Child span linked to parent
- Same trace ID maintained

---
##  What Happens Internally?
- App1 receives request → creates span
- Calls App2 → propagates trace header
- App2 creates new span (child)
- All spans → sent to Jaeger

---
##  Visualization (Jaeger)

You see:
- Full request flow
- Parent-child relationships
- Time taken per service

---
##  Key Takeaways
- Logs = detailed events
- Tracing = request journey
- Trace ID connects everything
- OpenTelemetry = modern standard
- Micrometer integrates easily with Spring Boot
