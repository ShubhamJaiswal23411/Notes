
# 1. Why Executor Framework Exists

Before Java 5, multithreading was done using:

```java
Thread t = new Thread(() -> {
    System.out.println("Task running");
});
t.start();
```

### Problems with Manual Threads

|Problem|Explanation|
|---|---|
|Thread creation cost|Creating threads is expensive|
|No thread reuse|Threads die after work|
|Hard to manage lifecycle|Need manual start/stop|
|No queueing|Cannot easily manage task backlog|
|Poor scalability|Too many threads → context switching|

### Solution

Java introduced **Executor Framework** in **Java 5 (`java.util.concurrent`)**

It separates:
```
Task submission
    ↓
Task execution
```

---
# 2. Executor (Core Interface)

## Definition
`Executor` is the **simplest interface** that represents an object capable of executing submitted tasks.
```java
public interface Executor {
    void execute(Runnable command);
}
```

### Key Idea
You submit a task → executor decides **how and when** to run it.

---
## Example
```java
Executor executor = new Executor() {
    public void execute(Runnable command) {
        new Thread(command).start();
    }
};

executor.execute(() -> System.out.println("Running task"));
```
### Output
```
Running task
```

But this **still creates a new thread each time**, so real implementations use **thread pools**.

---
## Task Types

Two common task types:
### Runnable
```java
Runnable task = () -> System.out.println("Task executed");
```

No return value.

---
### Callable
```java
Callable<Integer> task = () -> 10 + 20;
```

Returns result and can throw exception.

---
# 3. ExecutorService

`ExecutorService` extends `Executor` and adds **lifecycle management + advanced task control**.

```java
public interface ExecutorService extends Executor
```

---
## Key Capabilities

| Feature             | Method               |
| ------------------- | -------------------- |
| Submit task         | `submit()`           |
| Shutdown executor   | `shutdown()`         |
| Force shutdown      | `shutdownNow()`      |
| Wait for completion | `awaitTermination()` |
| Get result          | `Future`             |

---
## Example
```java
ExecutorService executor = Executors.newFixedThreadPool(3);

executor.submit(() -> {
    System.out.println("Task executed by " + Thread.currentThread().getName());
});

executor.shutdown();
```

---
## Using Callable + Future

```java
ExecutorService executor = Executors.newSingleThreadExecutor();

Callable<Integer> task = () -> 5 + 5;

Future<Integer> future = executor.submit(task);

Integer result = future.get();

System.out.println(result);

executor.shutdown();
```
### Output
```
10
```

---
## Future Interface
Represents **result of async computation**.
Important methods:

|Method|Purpose|
|---|---|
|`get()`|wait for result|
|`isDone()`|check completion|
|`cancel()`|cancel task|

---
# 4. ScheduledExecutorService

Used for **delayed and periodic tasks**.
It extends `ExecutorService`.
```java
public interface ScheduledExecutorService extends ExecutorService
```

---
## Common Methods

|Method|Purpose|
|---|---|
|`schedule()`|run after delay|
|`scheduleAtFixedRate()`|periodic execution|
|`scheduleWithFixedDelay()`|periodic after previous completion|

---

## Example — Delayed Task

```java
ScheduledExecutorService scheduler =
        Executors.newScheduledThreadPool(1);

scheduler.schedule(() -> {
    System.out.println("Executed after delay");
}, 3, TimeUnit.SECONDS);
```

---

## Fixed Rate Example

```java
scheduler.scheduleAtFixedRate(() -> {
    System.out.println("Running periodically");
}, 0, 5, TimeUnit.SECONDS);
```

Runs:

```
0s
5s
10s
15s
```

---

## Fixed Delay Example

```java
scheduler.scheduleWithFixedDelay(() -> {
    System.out.println("Running task");
}, 0, 5, TimeUnit.SECONDS);
```

Difference:

```
Task finishes → wait 5s → next execution
```

---
# 5. ThreadPoolExecutor (Real Implementation)

Most executors internally use:
```
ThreadPoolExecutor
```

Core architecture:
```
Task Queue
     ↓
Thread Pool
     ↓
Worker Threads execute tasks
```

---
## Constructor

```java
ThreadPoolExecutor(
 corePoolSize,
 maximumPoolSize,
 keepAliveTime,
 TimeUnit,
 BlockingQueue<Runnable>
)
```

---
## Example

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(
        2,
        4,
        60,
        TimeUnit.SECONDS,
        new LinkedBlockingQueue<>()
);
```

---
## How Tasks Are Handled
Order:
1. If running threads < `corePoolSize` → create new thread
2. Else → add task to queue
3. If queue full → create new thread up to `maximumPoolSize`
4. If still full → reject task

---
# 6. Executors Utility Class

Java provides **factory methods** to create thread pools.
```java
Executors
```
---
## 6.1 Fixed Thread Pool

```java
ExecutorService executor = Executors.newFixedThreadPool(4);
```
### Characteristics

| Property | Value           |
| -------- | --------------- |
| Threads  | Fixed           |
| Queue    | Unbounded       |
| Best for | CPU bound tasks |

---
### Example
```java
ExecutorService executor = Executors.newFixedThreadPool(2);

for(int i=0;i<5;i++){
    int task = i;
    executor.submit(() ->
        System.out.println("Task " + task + " executed by " +
        Thread.currentThread().getName())
    );
}

executor.shutdown();
```

---
## 6.2 Cached Thread Pool

```java
ExecutorService executor = Executors.newCachedThreadPool();
```
### Characteristics

| Property | Value             |
| -------- | ----------------- |
| Threads  | Dynamic           |
| Queue    | None              |
| Best for | Short-lived tasks |

Threads are reused but created as needed.

---
### Example
```java
ExecutorService executor = Executors.newCachedThreadPool();

for(int i=0;i<10;i++){
    executor.submit(() ->
        System.out.println(Thread.currentThread().getName())
    );
}

executor.shutdown();
```

---
# 6.3 Single Thread Executor

```java
ExecutorService executor = Executors.newSingleThreadExecutor();
```
### Characteristics

| Property | Value                |
| -------- | -------------------- |
| Threads  | 1                    |
| Ordering | Guaranteed           |
| Use case | sequential execution |

---
### Example
```java
ExecutorService executor = Executors.newSingleThreadExecutor();

executor.submit(() -> System.out.println("Task1"));
executor.submit(() -> System.out.println("Task2"));
executor.submit(() -> System.out.println("Task3"));

executor.shutdown();
```

Output:
```
Task1
Task2
Task3
```

---
## 6.4 Scheduled Thread Pool

```java
ScheduledExecutorService executor =
    Executors.newScheduledThreadPool(2);
```
Used for **cron-like scheduling**.

Example:
```java
executor.schedule(() -> {
    System.out.println("Delayed execution");
}, 2, TimeUnit.SECONDS);
```

---
## 6.5 Work Stealing Pool (Java 8)
```java
ExecutorService executor =
    Executors.newWorkStealingPool();
```
Uses **ForkJoinPool**.
### Characteristics

| Feature       | Explanation              |
| ------------- | ------------------------ |
| Work stealing | idle threads steal tasks |
| Parallelism   | high                     |
| Best for      | divide & conquer tasks   |

---
# 7. Advantages of Executor Framework

---
## 1 Thread Reuse
Instead of:
```
create thread
run task
destroy thread
```

Executors reuse threads.

---
## 2 Resource Management

Controls:
```
max threads
task queue
task rejection
```

---
## 3 Task Decoupling

Separates:
```
task submission
execution strategy
```

---
## 4 Scalability
Thread pools handle large numbers of tasks efficiently.

---
## 5 Built-in Scheduling
Using:
```
ScheduledExecutorService
```

---
## 6 Better Error Handling
Via:
```
Future
Callable
ExecutionException
```

---
# 8. Executor Framework Architecture
```
                 Executor
                    │
             ExecutorService
                    │
       ScheduledExecutorService
                    │
           ThreadPoolExecutor
                    │
            ScheduledThreadPoolExecutor
```

---
# 9. Best Practices
### Always Shutdown Executors
Bad
```
ExecutorService executor = Executors.newFixedThreadPool(2);
```

Good
```java
executor.shutdown();
executor.awaitTermination(10, TimeUnit.SECONDS);
```

---
### Avoid Executors.newFixedThreadPool in Production

Reason:
Unbounded queue → memory risk.

Prefer:
```java
new ThreadPoolExecutor(...)
```

---
### Choose Pool Size
Rule of thumb:
CPU bound:
```
threads = cores + 1
```

IO bound:
```
threads = cores * 2
```

---
# 10. Tricky Interview Questions

---
## Q1 Why is Executors class discouraged in production?
Answer:
Factory methods create thread pools with **unbounded queues**, which can lead to **OutOfMemoryError**.

Example:
```
newFixedThreadPool → LinkedBlockingQueue (unbounded)
```

Better:
```
ThreadPoolExecutor with bounded queue
```

## Q2 Difference Between execute() and submit()

| execute             | submit                      |
| ------------------- | --------------------------- |
| From Executor       | From ExecutorService        |
| Runnable only       | Runnable or Callable        |
| No return value     | returns Future              |
| Exceptions uncaught | exceptions stored in Future |

## Q3 Difference Between scheduleAtFixedRate vs scheduleWithFixedDelay

|Method|Behavior|
|---|---|
|Fixed Rate|runs every fixed interval|
|Fixed Delay|waits after previous execution|

Example timeline:
Fixed Rate
```
0s 5s 10s 15s
```

Fixed Delay
```
task → wait → task → wait
```

## Q4 What Happens When ThreadPool Queue Is Full?

Depends on **RejectedExecutionHandler**
Default:
```
AbortPolicy → throws RejectedExecutionException
```

Other policies:
```
CallerRunsPolicy
DiscardPolicy
DiscardOldestPolicy
```

## Q5 Difference Between newSingleThreadExecutor and newFixedThreadPool(1)

Tricky.

```
newSingleThreadExecutor
```
Guarantees **no reconfiguration**.

But:
```
newFixedThreadPool(1)
```
Can be cast and modified.

## Q6 What Is Work Stealing?

Threads with **empty queues steal tasks from busy threads**.
Used in:
```
ForkJoinPool
newWorkStealingPool()
```
Improves CPU utilization.

## Q7 Why Callable Instead of Runnable?
Runnable:
```
void run()
```

Callable:
```
V call() throws Exception
```

Advantages:
```
returns value
supports exceptions
```

# 11. Quick Summary

Executor Framework components:

```
Executor
ExecutorService
ScheduledExecutorService
ThreadPoolExecutor
Executors utility class
Future
Callable
```

Thread pool types:
```
FixedThreadPool
CachedThreadPool
SingleThreadExecutor
ScheduledThreadPool
WorkStealingPool
```

---

If you'd like, I can also create **another Obsidian note covering the INTERNALS of ThreadPoolExecutor (which is a VERY common senior interview question)** including:

- worker threads
    
- task queue types
    
- rejection policies
     
- lifecycle states
    
- how execute() works internally
    

That topic alone is **one of the most asked Java concurrency interview topics.**