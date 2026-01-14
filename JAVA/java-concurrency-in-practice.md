# Chapter 1 – Introduction to Concurrency

Writing correct concurrent programs is significantly harder than writing sequential ones because many more things can go wrong due to unpredictable interactions between threads. Despite this complexity, concurrency is essential in Java: threads simplify asynchronous programming, improve system responsiveness, and allow applications to fully utilize modern multiprocessor hardware.

---

## Why Concurrency Exists

Concurrency emerged to address three main problems:

- **Resource utilization**: While one task waits (e.g., for I/O), others can run.
- **Fairness**: Multiple users or programs can share CPU time.
- **Convenience**: Complex systems can be decomposed into cooperating tasks.

Early systems relied on **processes**, which are isolated programs with limited communication mechanisms. **Threads** evolved as a lighter-weight alternative, allowing multiple execution paths within a single process while sharing memory and resources.

---

## Threads and the Sequential Model

Most programming languages, including Java, follow a sequential model that is intuitive and human-friendly. Threads preserve this mental model by allowing developers to write mostly sequential logic while still handling asynchronous events.

However, because threads execute concurrently and share memory, incorrect coordination can lead to subtle and dangerous bugs.

---

## Benefits of Threads

Threads provide several major advantages:

### 1. Exploiting Multiple Processors
Multithreaded programs can run in parallel on multiple CPUs, dramatically improving throughput. A single-threaded program can only use one processor, wasting available computing power.

### 2. Simpler Modeling
Assigning threads to tasks creates the illusion of sequential execution and reduces complexity. Developers can focus on domain logic instead of scheduling, interleaving, and asynchronous control flow.

### 3. Handling Asynchronous Events
Blocking I/O becomes manageable when each task has its own thread. While one thread waits, others can continue making progress.

### 4. Responsive User Interfaces
Long-running tasks can run in background threads, keeping the UI responsive. This prevents user interfaces from freezing during expensive operations.

Threads are foundational to many Java frameworks, including servlets, RMI, Swing, and garbage collection, making concurrency unavoidable in real-world applications.

---

## Risks of Threads

Concurrency introduces three major categories of risk:

### 1. Safety Hazards
Without proper synchronization, threads may interfere with each other, leading to **race conditions**. Even simple operations (like `value++`) are not atomic. Java provides synchronization mechanisms to ensure predictable behavior, but developers must use them correctly.

### 2. Liveness Hazards
Threads can become permanently unable to make progress, resulting in:
- **Deadlock**
- **Starvation**
- **Livelock**

These bugs are often timing-dependent and difficult to reproduce.

### 3. Performance Hazards
Threads introduce overhead such as:
- Context switching
- Synchronization costs
- Cache invalidation

Poorly designed concurrency can degrade performance rather than improve it.

---

## Threads Are Everywhere

Even if an application does not explicitly create threads, frameworks do. Components invoked by:

- **Timers**
- **Servlets and JSPs**
- **Remote Method Invocation (RMI)**
- **Swing and AWT**

are executed in framework-managed threads.

This makes **thread safety contagious**: once shared state is accessed by multiple threads, all related code paths must be thread-safe.

---

## Core Message

Concurrency is powerful but dangerous. Because threads are pervasive in Java, developers must understand concurrency fundamentals to write safe, responsive, and scalable applications. Proper synchronization, careful design, and awareness of concurrency risks are essential.
