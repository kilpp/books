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


# Chapter 5: Building Blocks - Summary

## Overview
This chapter covers concurrent building blocks in the Java platform libraries, including thread-safe collections and synchronizers that coordinate control flow between threads. These components enable building robust concurrent applications through delegation and composition.

---

## 5.1 Synchronized Collections

### What They Are:
- **Legacy classes**: `Vector`, `Hashtable` (original JDK)
- **Wrapper classes**: `Collections.synchronizedXxx()` factory methods (JDK 1.2)
- **Thread safety mechanism**: Synchronize every public method with intrinsic lock

### 5.1.1 Problems with Synchronized Collections

#### Compound Actions Need Extra Locking
Common compound actions that are **not atomic** by default:
- **Iteration** - repeatedly fetch elements
- **Navigation** - find next element
- **Conditional operations** - put-if-absent

**Example Problem**: `getLast()` and `deleteLast()`
```java
// UNSAFE - check-then-act race condition
public static Object getLast(Vector list) {
    int lastIndex = list.size() - 1;  // Check
    return list.get(lastIndex);        // Act - may throw ArrayIndexOutOfBoundsException
}
```

**Solution**: Client-side locking
```java
// SAFE - synchronized on the collection
public static Object getLast(Vector list) {
    synchronized (list) {
        int lastIndex = list.size() - 1;
        return list.get(lastIndex);
    }
}
```

#### Iteration Problems
```java
// UNSAFE - can throw ArrayIndexOutOfBoundsException
for (int i = 0; i < vector.size(); i++)
    doSomething(vector.get(i));

// SAFE - lock held during entire iteration
synchronized (vector) {
    for (int i = 0; i < vector.size(); i++)
        doSomething(vector.get(i));
}
```

**Tradeoffs**:
- ✓ Prevents `ArrayIndexOutOfBoundsException`
- ✗ Hurts scalability (blocks all access during iteration)
- ✗ Risk of deadlock if `doSomething()` holds other locks
- **Alternative**: Clone collection and iterate the copy

### 5.1.2 Iterators and ConcurrentModificationException

**Fail-fast iterators**:
- Throw `ConcurrentModificationException` if collection modified during iteration
- Use modification count (checked without synchronization)
- "Good-faith effort" early warning, not foolproof

```java
// Can throw ConcurrentModificationException
List<Widget> widgetList = Collections.synchronizedList(new ArrayList<>());
for (Widget w : widgetList)
    doSomething(w);
```

**Solution**: Lock collection during iteration or clone it first

### 5.1.3 Hidden Iterators

⚠️ **Watch out for**: Methods that implicitly iterate:
- `toString()`, `hashCode()`, `equals()`
- `containsAll()`, `removeAll()`, `retainAll()`
- Collection constructors that take collections as arguments

```java
// UNSAFE - toString() iterates the set
System.out.println("DEBUG: added ten elements to " + set);
```

**Lesson**: Encapsulate synchronization to prevent these errors

---

## 5.2 Concurrent Collections

### Key Improvements over Synchronized Collections:
- **Better concurrency**: Multiple threads can access simultaneously
- **Better scalability**: Designed for concurrent access
- **No ConcurrentModificationException**: Weakly consistent iterators

### Main Classes (Java 5.0+):

| Class | Replaces | Use Case |
|-------|----------|----------|
| `ConcurrentHashMap` | `Hashtable`, `synchronizedMap` | Hash-based map with better concurrency |
| `CopyOnWriteArrayList` | `synchronizedList` | Traversal-dominant scenarios |
| `CopyOnWriteArraySet` | `synchronizedSet` | Traversal-dominant scenarios |
| `ConcurrentLinkedQueue` | - | Non-blocking FIFO queue |
| `BlockingQueue` | - | Producer-consumer designs |
| `ConcurrentSkipListMap` | `synchronizedSortedMap` | Concurrent sorted map (Java 6) |
| `ConcurrentSkipListSet` | `synchronizedSortedSet` | Concurrent sorted set (Java 6) |

### 5.2.1 ConcurrentHashMap

**Key Features**:
- **Lock striping**: Finer-grained locking for better concurrency
- **Multiple readers**: Arbitrary number of concurrent readers
- **Limited writers**: Some concurrent writers allowed
- **Weakly consistent iterators**: Tolerate concurrent modification
- **Result**: Far higher throughput with minimal single-thread penalty

**Tradeoffs**:
- `size()` and `isEmpty()` return approximations (moving targets)
- Cannot lock map for exclusive access
- ✓ Worth it: Better scalability for most use cases

**When NOT to use**: 
- Need to lock map for exclusive access
- Need exact `size()` values

### 5.2.2 Additional Atomic Map Operations

**`ConcurrentMap` interface** provides atomic compound operations:
```java
V putIfAbsent(K key, V value)
boolean remove(K key, V value)
boolean replace(K key, V oldValue, V newValue)
V replace(K key, V newValue)
```

**Benefit**: Eliminates need for client-side locking

### 5.2.3 CopyOnWriteArrayList

**How it works**:
- Creates new copy of backing array on every modification
- Iterators retain reference to array as it existed at creation time
- No synchronization needed during iteration
- Never throws `ConcurrentModificationException`

**When to use**:
- ✓ Iteration far more common than modification
- ✓ Event-notification systems (listener lists)
- ✗ Large collections with frequent modifications

---

## 5.3 Blocking Queues and Producer-Consumer Pattern

### BlockingQueue Operations:
- `put()` - blocks if queue is full
- `take()` - blocks if queue is empty
- `offer()` - timed version of put, returns failure status
- `poll()` - timed version of take

### Producer-Consumer Pattern Benefits:
1. **Decoupling**: Producers don't know about consumers and vice versa
2. **Flexible workload management**: Handle different production/consumption rates
3. **Simplified development**: Remove code dependencies
4. **Built-in flow control**: Blocking naturally handles overload

### Implementations:

| Implementation | Description |
|----------------|-------------|
| `LinkedBlockingQueue` | FIFO queue (like `LinkedList`) |
| `ArrayBlockingQueue` | FIFO queue (like `ArrayList`) |
| `PriorityBlockingQueue` | Priority-ordered queue |
| `SynchronousQueue` | No storage - direct handoff |

### SynchronousQueue:
- Maintains no storage, only queued threads
- `put()` blocks until another thread calls `take()`
- Direct handoff reduces latency
- Provides feedback to producer about consumption
- Suitable when enough consumers available

### 5.3.1 Example: Desktop Search
**Pattern**: File crawler (producer) + File indexer (consumer)
- Crawler finds files, puts on queue
- Indexer takes files from queue, indexes them
- Better performance: I/O-bound crawler + CPU-bound indexer run concurrently

### 5.3.2 Serial Thread Confinement
**Concept**: Transfer ownership of confined object between threads
- Object owned exclusively by one thread at a time
- Safe publication via `BlockingQueue` ensures visibility
- Original owner must not access after handoff
- **Examples**: Object pools, work queues

### 5.3.3 Deques and Work Stealing
**`Deque`** (Java 6): Double-ended queue
- Efficient insertion/removal from both ends
- Implementations: `ArrayDeque`, `LinkedBlockingDeque`

**Work Stealing Pattern**:
- Each consumer has own deque
- When deque empty, "steal" from tail of another's deque
- **Benefits**: Less contention, more scalable
- **Use cases**: Recursive problems (web crawling, graph exploration, GC marking)

---

## 5.4 Blocking and Interruptible Methods

### Thread States:
- **BLOCKED**: Waiting to acquire lock
- **WAITING**: Waiting indefinitely for event
- **TIMED_WAITING**: Waiting with timeout

### InterruptedException:
**Signals**: Method is blocking and responsive to interruption

**Two proper responses**:

#### 1. Propagate the exception
```java
// Let it bubble up - simplest approach
public void method() throws InterruptedException {
    queue.take();
}
```

#### 2. Restore interrupted status
```java
// When can't throw (e.g., Runnable)
public void run() {
    try {
        processTask(queue.take());
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt(); // Restore status
    }
}
```

⚠️ **NEVER**: Catch and swallow interrupt without restoring status
- Exception: When extending `Thread` and controlling entire stack

---

## 5.5 Synchronizers

**Definition**: Coordinate control flow of threads based on state

### 5.5.1 Latches (CountDownLatch)

**Characteristics**:
- Single-use gate (cannot be reset)
- Blocks threads until terminal state reached
- Terminal state = gate opens forever

**Common uses**:
1. Ensure resources initialized before computation
2. Ensure dependencies started before service starts
3. Wait for all parties ready (multiplayer game)

**Example**: `TestHarness` with starting/ending gates
```java
CountDownLatch startGate = new CountDownLatch(1);
CountDownLatch endGate = new CountDownLatch(nThreads);

// Workers wait on start gate
startGate.await();
// ... do work ...
endGate.countDown();

// Master releases all workers simultaneously
startGate.countDown();
// Master waits for all to complete
endGate.await();
```

### 5.5.2 FutureTask

**States**: Waiting to run → Running → Completed
- Completed includes: normal, cancellation, exception
- Once completed, stays completed forever

**Behavior**:
- `get()` returns immediately if completed
- `get()` blocks if not completed
- Safe publication of result guaranteed

**Use case**: Preloading expensive data
```java
FutureTask<ProductInfo> future = new FutureTask<>(() -> loadProductInfo());
Thread thread = new Thread(future);
thread.start(); // Start loading early

// Later...
ProductInfo info = future.get(); // Returns immediately if ready, else waits
```

### 5.5.3 Semaphores

**Purpose**: Control number of activities accessing resource/performing action

**How it works**:
- Manages set of virtual permits
- `acquire()` - blocks until permit available
- `release()` - returns permit to semaphore
- Binary semaphore (count=1) = non-reentrant mutex

**Use cases**:
- **Resource pools**: Database connection pools
- **Bounded collections**: Turn any collection into blocking bounded collection

**Example**: `BoundedHashSet`
```java
Semaphore sem = new Semaphore(bound);

public boolean add(T o) throws InterruptedException {
    sem.acquire();
    boolean wasAdded = false;
    try {
        wasAdded = set.add(o);
        return wasAdded;
    } finally {
        if (!wasAdded) sem.release(); // Return permit if add failed
    }
}
```

### 5.5.4 Barriers (CyclicBarrier)

**Difference from latches**:
- **Latches**: Wait for events
- **Barriers**: Wait for other threads

**CyclicBarrier**:
- All threads must reach barrier point together
- Can be reused (cyclic)
- Supports barrier action (runs when barrier passed)
- Returns arrival index for each thread

**Use cases**:
- Parallel iterative algorithms
- Simulations (e.g., cellular automata, n-body particle simulations)
- Step k must complete before step k+1 begins

**Example**: Cellular automata
```java
CyclicBarrier barrier = new CyclicBarrier(nWorkers, 
    () -> mainBoard.commitNewValues());

// In each worker:
while (!board.hasConverged()) {
    // Calculate new values for partition
    barrier.await(); // Wait for all workers, then commit
}
```

**Exchanger**:
- Two-party barrier for exchanging data
- Safe publication to both parties
- Use case: One thread fills buffer, another consumes it

---

## 5.6 Building an Efficient, Scalable Result Cache

### Evolution of Memoizer:

#### Memoizer1 - HashMap + synchronized
```java
public synchronized V compute(A arg) {
    V result = cache.get(arg);
    if (result == null) {
        result = c.compute(arg);
        cache.put(arg, result);
    }
    return result;
}
```
- ✗ **Problem**: Only one thread can execute at a time
- ✗ **Result**: Poor concurrency, possible worse than no cache

#### Memoizer2 - ConcurrentHashMap
```java
public V compute(A arg) {
    V result = cache.get(arg);
    if (result == null) {
        result = c.compute(arg);
        cache.put(arg, result);
    }
    return result;
}
```
- ✓ Better concurrency
- ✗ **Problem**: Two threads can compute same value (check-then-act race)

#### Memoizer3 - ConcurrentHashMap + FutureTask
```java
ConcurrentHashMap<A, Future<V>> cache;

Future<V> f = cache.get(arg);
if (f == null) {
    FutureTask<V> ft = new FutureTask<>(() -> c.compute(arg));
    f = ft;
    cache.put(arg, ft);
    ft.run();
}
return f.get();
```
- ✓ Much better: Threads wait for in-progress computations
- ✗ **Problem**: Small window where two threads can still compute same value

#### Memoizer (Final) - putIfAbsent
```java
while (true) {
    Future<V> f = cache.get(arg);
    if (f == null) {
        FutureTask<V> ft = new FutureTask<>(() -> c.compute(arg));
        f = cache.putIfAbsent(arg, ft); // Atomic!
        if (f == null) { f = ft; ft.run(); }
    }
    try {
        return f.get();
    } catch (CancellationException e) {
        cache.remove(arg, f); // Remove cancelled computations
    }
}
```
- ✓ **Perfect**: Atomic put-if-absent eliminates race condition
- ✓ Handles cancellation (cache pollution)
- Future improvements: Cache expiration, eviction

---

## Summary of Part I: Concurrency Cheat Sheet

### Core Principles:

1. **It's the mutable state, stupid**
   - All concurrency issues = coordinating access to mutable state
   - Less mutable state = easier thread safety

2. **Make fields final unless they need to be mutable**

3. **Immutable objects are automatically thread-safe**
   - Simpler, safer, can be shared freely without locking

4. **Encapsulation makes it practical to manage complexity**
   - Encapsulate data to preserve invariants
   - Encapsulate synchronization to comply with policy

5. **Guard each mutable variable with a lock**

6. **Guard all variables in an invariant with the same lock**

7. **Hold locks for the duration of compound actions**

8. **A program accessing mutable variables from multiple threads without synchronization is broken**

9. **Don't rely on clever reasoning about why you don't need to synchronize**

10. **Include thread safety in design OR document that class is not thread-safe**

11. **Document your synchronization policy**

---

## Key Takeaways

### Collections:
- **Synchronized collections**: Legacy, use concurrent collections instead
- **ConcurrentHashMap**: Almost always better than `Hashtable`/`synchronizedMap`
- **CopyOnWriteArrayList**: Great for listener lists (read-heavy)
- **Lock during iteration OR clone**: For synchronized collections

### Blocking Queues:
- **Producer-consumer**: Natural fit for blocking queues
- **Bounded queues**: Essential for resource management and robustness
- **Work stealing**: Scalable alternative to shared work queue

### Synchronizers:
- **Latches**: One-time events (starting gun)
- **Barriers**: Recurring synchronization points (rendezvous)
- **Semaphores**: Resource pools, bounded collections
- **FutureTask**: Deferred computation results

### Best Practices:
- Prefer concurrent collections over synchronized collections
- Use blocking queues to simplify producer-consumer designs
- Build resource management in early (bounded queues)
- Choose right synchronizer for the problem
- Document thread safety guarantees

---

## Quick Reference

### When to Use What:

| Need | Use |
|------|-----|
| Thread-safe map | `ConcurrentHashMap` |
| Thread-safe list (read-heavy) | `CopyOnWriteArrayList` |
| Producer-consumer | `BlockingQueue` |
| Resource pool | `Semaphore` + bounded structure |
| One-time initialization | `CountDownLatch` or `FutureTask` |
| Iterative parallel algorithm | `CyclicBarrier` |
| Deferred computation | `FutureTask` |
| Result caching | `ConcurrentHashMap` + `FutureTask` |

### Performance Tips:
- Iteration on synchronized collections: Consider cloning if iteration frequent
- Large `CopyOnWriteArrayList`: Only if reads >> writes
- `ConcurrentHashMap.size()`: Returns approximation, don't rely on exactness
- Work stealing: Better than shared queue for recursive problems


