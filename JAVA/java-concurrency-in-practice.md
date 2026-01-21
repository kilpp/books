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

### **Chapter 2: Thread Safety**

This chapter establishes the fundamental concepts of writing thread-safe code, focusing on managing access to shared, mutable state.

* **Definition of Thread Safety**: A class is thread-safe if it behaves correctly when accessed from multiple threads, regardless of their scheduling or interleaving, without requiring additional synchronization by the calling code.
* **Atomicity and Race Conditions**: 
    * **Race Conditions**: Occur when the correctness of a computation depends on the relative timing of multiple threads (e.g., "check-then-act").
    * **Atomic Operations**: Operations that appear to happen all at once. To avoid race conditions, compound actions (like incrementing a counter) must be made atomic.
* **Locking**: 
    * **Intrinsic Locks**: Java provides the `synchronized` block as a built-in locking mechanism. These locks are **reentrant**, meaning a thread can acquire a lock it already holds.
    * **Guarding State**: For every mutable state variable accessed by more than one thread, all accesses to that variable must be performed with the same lock held.
* **Liveness and Performance**: Excessive synchronization can lead to performance bottlenecks. The goal is to guard the state without unnecessarily restricting concurrent access to tasks that don't transition shared state.

---

### **Chapter 3: Sharing Objects**

This chapter shifts focus from atomicity to the **visibility** and **publication** of objects to ensure that when one thread modifies the state, other threads actually see the change.

* **Visibility**: Without synchronization, the compiler, hardware, and runtime may reorder operations. This can lead to threads seeing "stale data."
* **Volatile Variables**: A weaker form of synchronization. Marking a field as `volatile` ensures that updates to the variable are propagated predictably to other threads, though it does not provide atomicity for compound actions.
* **Publication and Escape**:
    * **Publication**: Making an object available outside its current scope (e.g., returning a reference from a getter).
    * **Escape**: When an object is published that should have been internal. An object that escapes during construction can lead to thread-safety issues.
* **Thread Confinement**: A simple way to achieve thread safety by not sharing data. Types include **Ad-hoc thread confinement**, **Stack confinement** (local variables), and `ThreadLocal`.
* **Immutability**: Immutable objects are inherently thread-safe because their state cannot change. They can be shared freely without synchronization.
* **Safe Publication**: To share an object safely, both the reference and the state must be made visible. This can be done through static initializers, `volatile` fields, `final` fields, or guarding the reference with a lock.

---
