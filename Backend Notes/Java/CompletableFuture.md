
##  1. Why CompletableFuture Exists

### Problems with older approaches:
- `Future` (Java 5):
    - Cannot chain
    -  Cannot combine multiple futures
    -  No callbacks
    -  Blocking (`get()`)

### Solution:
👉 `CompletableFuture` (Java 8)
- Combines:
    - `Future` (result holder)
    - `CompletionStage` (pipeline of async tasks)
    - get() exists here as well.

---
##  2. Core Concepts

### Dual Nature

```java
CompletableFuture<T>
```

Acts as:
1. A **container for result**
2. A **pipeline of dependent actions** 

---
### 1. Creation Methods

```java
CompletableFuture.supplyAsync(() -> "Hello");
CompletableFuture.runAsync(() -> System.out.println("Run"));
```

| Method        | Returns                   | Use           |
| ------------- | ------------------------- | ------------- |
| `runAsync`    | `CompletableFuture<Void>` | No result     |
| `supplyAsync` | `CompletableFuture<T>`    | Returns value |

---
##  3. Execution Threads (CRITICAL for Interviews)

### Default behavior:
- Uses **ForkJoinPool.commonPool()**
### Custom executor:

```java
ExecutorService executor = Executors.newFixedThreadPool(5);
CompletableFuture.supplyAsync(() -> "Hi", executor);
```

---
### 1. IMPORTANT BEHAVIOR

#### 1. Non-async methods:
- Run in **same thread that completes previous stage or the caller thread if the called thread is blocked**
#### 2. Async methods:
- Run in:
    - Provided executor OR
    - ForkJoinPool.commonPool()

---
##  4. Chaining Methods (CORE OF CF)

---
# 4.1 Transforming (Map-like)

---
##  thenApply

```java
cf.thenApply(x -> x + " World");
```
- Transforms result
- Returns new value
- Order is maintained

---
##  thenApplyAsync

```java
cf.thenApplyAsync(x -> x + " World");
```
### Difference:

| thenApply                  | thenApplyAsync                 |
| -------------------------- | ------------------------------ |
| Same thread                | Schedules from the thread pool |
| Faster (no context switch) | Safer for long tasks           |

---

###  Interview Insight:
- `thenApply()` may run in **main thread**, if the main is already blocked by .get() call.
- `thenApplyAsync()` guarantees **async execution**, main will not be blocked and the task will always be scheduled on a thread pool.

---
#  4.2 FlatMap-like (Dependent Async)

---
##  1. thenCompose

```java
cf.thenCompose(x -> CompletableFuture.supplyAsync(() -> x + " World"));
```
- Avoids `CompletableFuture<CompletableFuture<T>>`
- Used when function returns **`CompletableFuture<T>`**
- Prevents **nested futures**
- In the above example since the inside task also returns a CompletableFuture that would create a nested structure like this `CompletableFuture<CompletableFuture<T>>` and then in order to get the result we would have to do .get().get() so in order to flatten this structure we use thenCompose.

---
##  2. thenComposeAsync

Same async behavior applies

---
### 3. Difference: thenApply vs thenCompose

|thenApply|thenCompose|
|---|---|
|wraps result|flattens future|
|T → U|T → CompletableFuture|

---

# 4.3 Combining Two Futures

---

## 1. thenCombine

```java
cf1.thenCombine(cf2, (a, b) -> a + b);
```
- Waits for both
- Combines results
    

---

## 2. thenCombineAsync
Async version

---
## 3. thenAcceptBoth

```java
cf1.thenAcceptBoth(cf2, (a, b) -> System.out.println(a + b));
```
- Consumes result
- Returns `Void`

---
## 4. runAfterBoth

```java
cf1.runAfterBoth(cf2, () -> System.out.println("Done"));
```
- No inputs, no outputs

---

#  4.4 Race Conditions (First Completion)

---
## 1. applyToEither

```java
cf1.applyToEither(cf2, x -> x + " processed");
```
- Runs when **any one completes**

---
## 2. acceptEither
Consumes result

---
## 3. runAfterEither
No input/output

---

#  4.5 Consuming Results

---
## 1. thenAccept

```java
cf.thenAccept(x -> System.out.println(x));
```
- Takes input
- No return

---
## 2. thenRun

```java
cf.thenRun(() -> System.out.println("Done"));
```

- No input, no return

---
#  4.6 Exception Handling

---

## 1. exceptionally

```java
cf.exceptionally(ex -> "fallback");
```
- Recovers from failure

---
## 2. handle

```java
cf.handle((res, ex) -> {
    if (ex != null) return "error";
    return res;
});
```
- Always runs
- Gives both result + exception

---
## 3. whenComplete

```java
cf.whenComplete((res, ex) -> {
    System.out.println("Completed");
});
```
- Side-effect only
- Does NOT modify result

---
###  Difference:

|Method|Can change result?|Handles exception?|
|---|---|---|
|exceptionally|✔|✔|
|handle|✔|✔|
|whenComplete|❌|✔|

---
#  5. Async vs Non-Async (VERY IMPORTANT)

---

## 1. Rule of Thumb

| Method Type | Thread                                    |
| ----------- | ----------------------------------------- |
| Non-Async   | Same thread (or completing thread)        |
| Async       | Different thread(never the caller thread) |

---
### Example:

```java
CompletableFuture.supplyAsync(() -> "Hello")
    .thenApply(x -> x + " World")       // SAME thread that performed previous operation.
    .thenApplyAsync(x -> x + "!");      // NEW thread
```

---
###  Key Insight
- If previous stage completes in main thread → next non-async runs in main thread
- If previous stage completes in worker thread → next non-async runs there

---
###  Interview Insight

> ForkJoinPool is NOT ideal for blocking tasks (like I/O)

---

# 6. Internal Working (Advanced)

---
## Completion Queue
Each stage registers:
- A callback
- Triggered when previous completes

---
## Execution Model
- Tasks stored as **Completion objects**
- Triggered via **stack-like structure**

---
## ForkJoinPool Behavior
- Work-stealing
- Optimized for CPU-bound tasks

---
# 7. Common Patterns

---
## 1. Sequential Chain

```java
supplyAsync(...)
.thenApply(...)
.thenApply(...)
```

---
## 2. Parallel + Combine

```java
cf1.thenCombine(cf2, ...)
```

---
## 3. Race

```java
applyToEither
```

---
## 4. Error Recovery

```java
exceptionally
```

---
#  9. Common Pitfalls

---
-   Blocking inside async
```java
cf.thenApply(x -> blockingCall());
```
 BAD for common pool

- Forgetting async
```java
thenApply() // may block main thread
```

-  Nested futures
```java
thenApply(x -> CompletableFuture...)
```
 Use `thenCompose`

---
#  10. Tricky Interview Questions (with Answers)

---
##  1. Difference between thenApply and thenCompose?
`thenApply` wraps result  
`thenCompose` flattens nested futures

## 2. When does thenApply run in main thread?
If previous stage completes in main thread or if main thread is blocked by .get() call.

## 3. Does thenApplyAsync always create a new thread?
No , It Uses thread pool (may reuse threads)

## 4. What happens if you don’t provide executor?
Uses `ForkJoinPool.commonPool()`

## 5. Why is common pool dangerous?
Blocking tasks can:
- Starve threads
- Freeze pipeline
## 6. Difference between handle and exceptionally?
 `handle`:
- Always runs
- Has result + exception
`exceptionally`:
- Only runs on failure
## 7. thenAccept vs thenRun?

| Method     | Input | Output |
| ---------- | ----- | ------ |
| thenAccept | ✔     | ❌      |
| thenRun    | ❌     | ❌      |
## 8. What is CompletionStage?
Interface implemented by `CompletableFuture`  
Defines chaining methods
## 9. Can CompletableFuture be manually completed?

```java
cf.complete("value");
```
YES
## 10. Difference between join() and get()?

| Method | Checked Exception | Wraps exception    |
| ------ | ----------------- | ------------------ |
| get()  | ✔                 | ExecutionException |
| join() | ❌                 | RuntimeException   |

---
#  11. Mental Model (Best Way to Remember)

Think of CompletableFuture as:
> 🔗 A pipeline of transformations where each step decides:

- sync vs async    
- transformation vs consumption
- dependency vs independence

---

#  Final Cheat Sheet

---
### Transform:
- thenApply / thenCompose
### Combine:
- thenCombine / thenAcceptBoth
### Race:
- applyToEither
### Consume:
- thenAccept / thenRun
### Error:
- exceptionally / handle / whenComplete
