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

# Chapter 4: Composing Objects - Summary

## Overview
This chapter covers patterns for structuring thread-safe classes and safely composing them into larger components without undermining their safety guarantees.

---

## 4.1 Designing a Thread-safe Class

### Three Basic Elements:
1. **Identify the variables** that form the object's state
2. **Identify the invariants** that constrain the state variables
3. **Establish a policy** for managing concurrent access to the object's state

### Key Concepts:
- **Object State**: Comprises all fields, including primitive types and references to other objects
- **Synchronization Policy**: Defines how an object coordinates access to its state using:
  - Immutability
  - Thread confinement
  - Locking (which locks guard which variables)

### Important Requirements:

#### 4.1.1 Gathering Synchronization Requirements
- Understand the **state space** (range of possible states)
- Use `final` fields to simplify reasoning about possible states
- **Invariants** identify valid vs. invalid states
- **Post-conditions** identify valid vs. invalid state transitions
- Multivariable invariants require atomic operations across related variables

#### 4.1.2 State-dependent Operations
- Operations with **state-based preconditions** (e.g., can't remove from empty queue)
- Use built-in mechanisms (`wait`/`notify`) or library classes (`BlockingQueue`, `Semaphore`)
- Concurrent programs can wait for preconditions to become true

#### 4.1.3 State Ownership
- Ownership is a design element, not explicitly in the language
- Owner decides the locking protocol
- Publishing a mutable object loses exclusive control
- **Split ownership**: Collection owns infrastructure, client owns stored objects (e.g., `ServletContext`)

---

## 4.2 Instance Confinement

### Core Principle:
**Encapsulation + Locking = Thread Safety**

- Confine non-thread-safe objects within a thread-safe wrapper
- All access paths are known and can be analyzed
- Objects must not escape their intended scope

### 4.2.1 The Java Monitor Pattern
- Encapsulates all mutable state
- Guards state with the object's own intrinsic lock
- All methods accessing state are synchronized
- **Example**: `Counter` class

**Alternative**: Use a private lock object instead of intrinsic lock
- Prevents client code from participating in synchronization policy
- Simplifies verification (no need to examine entire program)

### 4.2.2 Example: Vehicle Tracker
**Monitor-based Implementation** (`MonitorVehicleTracker`):
- Uses non-thread-safe `MutablePoint` internally
- Maintains thread safety through confinement
- Returns defensive copies via `deepCopy()`
- Provides consistent snapshot but may have performance impact

---

## 4.3 Delegating Thread Safety

### When It Works:
- Delegate to thread-safe components when:
  - State variables are independent
  - No invariants couple multiple state variables
  - No invalid state transitions

### 4.3.1 Example: Delegating Vehicle Tracker
**`DelegatingVehicleTracker`**:
- Uses `ConcurrentHashMap` with immutable `Point` objects
- No explicit synchronization needed
- Returns "live" view (updates reflected immediately)
- Can return static copy if unchanging view needed

### 4.3.2 Independent State Variables
- Can delegate to multiple thread-safe variables if independent
- **Example**: `VisualComponent` with separate mouse and key listeners using `CopyOnWriteArrayList`

### 4.3.3 When Delegation Fails
- **Example**: `NumberRange` class (DO NOT USE)
  - Uses two `AtomicInteger` variables
  - Has invariant: `lower <= upper`
  - `setLower()` and `setUpper()` are check-then-act sequences
  - Not atomic, can violate invariant
  - **Solution**: Use locking to maintain invariants

### 4.3.4 Publishing Underlying State Variables
**Safe to publish if**:
- State variable is thread-safe
- Does not participate in invariants
- Has no prohibited state transitions

### 4.3.5 Example: Publishing Vehicle Tracker
**`PublishingVehicleTracker`**:
- Uses `ConcurrentHashMap` with thread-safe `SafePoint`
- Publishes mutable state safely
- Clients can modify vehicle locations directly
- `SafePoint` provides atomic getter for both coordinates

---

## 4.4 Adding Functionality to Existing Thread-safe Classes

### Three Approaches:

#### 1. **Modify the Original Class**
- Safest approach if you have access to source code
- Keeps synchronization policy in one place

#### 2. **Extend the Class**
- **Example**: `BetterVector extends Vector`
- More fragile - synchronization policy distributed across files
- Only works if class designed for extension
- Risk: Subclass breaks if superclass changes synchronization policy

#### 3. **Client-side Locking** (FRAGILE)
- Use the same lock the class uses
- **Example**: `synchronized(list) { ... }`
- Very fragile - couples to implementation details
- Violates encapsulation of synchronization policy

#### 4. **Composition** (RECOMMENDED)
- **Example**: `ImprovedList` wrapper
- Implements interface, delegates to underlying instance
- Provides own consistent locking layer
- Less fragile, guaranteed thread-safe
- Small performance penalty but much safer

---

## 4.5 Documenting Synchronization Policies

### Critical Documentation:
**For clients**:
- Is the class thread-safe?
- Thread safety guarantees
- Which locks to acquire for new atomic operations
- Does it make callbacks with lock held?

**For maintainers**:
- Which variables are `@GuardedBy` which locks
- Synchronization strategy (immutability, confinement, locking)
- Which operations must be atomic

### 4.5.1 Interpreting Vague Documentation
When specifications are unclear (e.g., Servlet, JDBC):
- Consider implementer's perspective
- Container-managed objects accessed by multiple threads likely thread-safe
- Examples in specs usually indicate expected usage
- Objects you store in containers (e.g., `HttpSession` attributes) should be thread-safe

---

## Key Takeaways

1. **Design for thread safety from the start** - identify state, invariants, and synchronization policy
2. **Encapsulation is your friend** - confine state and control access
3. **Delegation works when variables are independent** - otherwise, add your own locking
4. **Composition over extension** - safest way to add functionality
5. **Document everything** - thread safety guarantees and synchronization policy
6. **Don't guess** - if unclear, assume you need to provide thread safety

---

## Common Patterns Summary

| Pattern | Use When | Example |
|---------|----------|---------|
| Java Monitor | Building from scratch | `Counter` |
| Instance Confinement | Wrapping non-thread-safe objects | `PersonSet` |
| Delegation | Components are independent | `DelegatingVehicleTracker` |
| Composition | Adding functionality safely | `ImprovedList` |

---

## Warning Signs

⚠️ **Avoid**:
- Publishing mutable state without proper guards
- Client-side locking (very fragile)
- Check-then-act without atomicity (`NumberRange` anti-pattern)
- Assuming classes are thread-safe without documentation
- Relying on vague specifications without verification


