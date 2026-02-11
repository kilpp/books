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

---

# Chapter 6: Task Execution - Summary

## Overview
This chapter explores organizing concurrent applications around **task execution**, moving from manual thread creation to the powerful **Executor framework**. Most concurrent applications are naturally structured around discrete units of work (tasks), and properly managing their execution is crucial for performance, responsiveness, and resource management.

---

## 6.1 Executing Tasks in Threads

### Identifying Task Boundaries
**Ideal tasks are**:
- **Independent**: Don't depend on state, results, or side effects of other tasks
- **Small**: Represent a small fraction of application's processing capacity
- **Natural units**: E.g., individual client requests (HTTP, mail, database queries)

**Benefits of good task boundaries**:
- ✓ Simplifies program organization
- ✓ Facilitates error recovery (natural transaction boundaries)
- ✓ Promotes concurrency through natural parallelization

---

### 6.1.1 Sequential Execution (SingleThreadWebServer)

```java
class SingleThreadWebServer {
    public static void main(String[] args) throws IOException {
        ServerSocket socket = new ServerSocket(80);
        while (true) {
            Socket connection = socket.accept();
            handleRequest(connection);  // Blocks everything!
        }
    }
}
```

**Problems**:
- ✗ **Poor responsiveness**: New connections wait while request is processed
- ✗ **Poor throughput**: CPU sits idle during I/O operations
- ✗ **Poor resource utilization**: Single thread can't utilize multiple processors
- ✗ **Blocking**: One slow request blocks all others

**When acceptable**: 
- Tasks are few and long-lived
- Server serves single client making one request at a time
- (Rare in real server applications)

---

### 6.1.2 Thread-Per-Task (ThreadPerTaskWebServer)

```java
class ThreadPerTaskWebServer {
    public static void main(String[] args) throws IOException {
        ServerSocket socket = new ServerSocket(80);
        while (true) {
            final Socket connection = socket.accept();
            Runnable task = new Runnable() {
                public void run() { handleRequest(connection); }
            };
            new Thread(task).start();  // Create new thread for each request
        }
    }
}
```

**Improvements**:
- ✓ **Better responsiveness**: Main thread resumes accepting connections immediately
- ✓ **Parallel processing**: Multiple requests handled simultaneously
- ✓ **Better throughput**: Especially on multiprocessor systems

**Requirements**:
- Task-handling code must be thread-safe (concurrent invocations)

---

### 6.1.3 Disadvantages of Unbounded Thread Creation

⚠️ **Major Problems**:

#### 1. Thread Lifecycle Overhead
- Thread creation and teardown consume significant resources
- Introduces latency into request processing
- For lightweight, frequent requests, overhead can dominate

#### 2. Resource Consumption
- **Active threads consume memory** (stacks, thread-local storage)
- Idle threads tie up memory and stress garbage collector
- Too many threads competing for CPUs → context switching overhead
- Beyond keeping CPUs busy, more threads = worse performance

#### 3. Stability Issues
- **Platform limits** on number of threads (OS, JVM, hardware dependent)
- Hitting limit typically → `OutOfMemoryError`
- Recovery from `OutOfMemoryError` is extremely risky
- On 32-bit JVMs: ~½ MB stack per thread → few thousand threads max

**Critical Failure Mode**:
- No inherent limit on thread creation
- Can crash under heavy load or malicious traffic
- Violates requirement for graceful degradation

**Lesson**: Must place bounds on thread creation

---

## 6.2 The Executor Framework

### Core Abstraction

```java
public interface Executor {
    void execute(Runnable command);
}
```

**Key innovation**: Decouples **task submission** from **task execution**

**Benefits**:
- Flexible execution policies (sequential, thread-per-task, thread pool, etc.)
- Standard mechanism for task submission
- Easier to change execution policy without changing task submission code
- Based on **producer-consumer pattern**: Tasks are produced, threads consume them

---

### 6.2.1 Example: Web Server Using Executor

```java
class TaskExecutionWebServer {
    private static final int NTHREADS = 100;
    private static final Executor exec = Executors.newFixedThreadPool(NTHREADS);
    
    public static void main(String[] args) throws IOException {
        ServerSocket socket = new ServerSocket(80);
        while (true) {
            final Socket connection = socket.accept();
            Runnable task = () -> handleRequest(connection);
            exec.execute(task);  // Submit to executor
        }
    }
}
```

**Advantages**:
- ✓ Bounds resource usage (max 100 threads)
- ✓ Won't crash under heavy load
- ✓ Easy to change behavior by swapping `Executor` implementation
- ✓ Graceful degradation under load

---

### 6.2.2 Execution Policies

**Defines the "what, where, when, and how" of task execution**:

1. **In what thread** will tasks execute?
2. **In what order** (FIFO, LIFO, priority)?
3. **How many tasks** may execute concurrently?
4. **How many tasks** may be queued pending execution?
5. **If overloaded**, which task to reject and how to notify?
6. **What actions** before/after task execution?

**Resource management tool**: Optimal policy depends on:
- Available computing resources (CPUs, memory)
- Quality-of-service requirements

⚠️ **Replacement pattern**: Whenever you see `new Thread(runnable).start()`, seriously consider using an `Executor` instead.

---

### 6.2.3 Thread Pools

**How they work**:
- Manage homogeneous pool of **worker threads**
- Workers request tasks from **work queue**, execute them, return for more
- Reuse existing threads instead of creating new ones

**Advantages over thread-per-task**:
- ✓ **Amortized creation costs**: Thread creation/teardown spread over many requests
- ✓ **Better responsiveness**: Worker often exists when request arrives (no creation latency)
- ✓ **Tunable**: Size pool to keep processors busy without exhausting resources

---

#### Standard Thread Pool Factories (Executors)

| Factory Method | Description | Use When |
|----------------|-------------|----------|
| `newFixedThreadPool(n)` | Fixed-size pool (n threads) | Known workload, want bounded resource usage |
| `newCachedThreadPool()` | Unbounded pool, reaps idle threads | Many short-lived tasks, bursty load |
| `newSingleThreadExecutor()` | Single worker thread | Need sequential execution (FIFO, LIFO, priority) |
| `newScheduledThreadPool(n)` | Fixed-size, supports delayed/periodic tasks | Like Timer but better |

**Configuration**:
- All return `ThreadPoolExecutor` instances
- Can construct `ThreadPoolExecutor` directly for specialized needs (Chapter 8)

**Effect on stability**:
- Fixed/single pools provide bounded resource usage
- Prevents failure under heavy load
- Enables tuning, monitoring, logging, error reporting

---

### 6.2.4 Executor Lifecycle

**Problem**: JVM can't exit until all non-daemon threads terminate

**ExecutorService interface** extends Executor with lifecycle methods:

```java
public interface ExecutorService extends Executor {
    void shutdown();                          // Graceful shutdown
    List<Runnable> shutdownNow();            // Abrupt shutdown
    boolean isShutdown();
    boolean isTerminated();
    boolean awaitTermination(long timeout, TimeUnit unit);
    // ... task submission convenience methods
}
```

#### Three States:
1. **Running**: Normal operation
2. **Shutting down**: No new tasks accepted, existing tasks complete
3. **Terminated**: All tasks completed

#### Shutdown Methods:

**`shutdown()`** - Graceful:
- No new tasks accepted
- Previously submitted tasks (including queued) allowed to complete

**`shutdownNow()`** - Abrupt:
- Attempts to cancel outstanding tasks
- Doesn't start queued tasks
- Returns list of tasks that were queued but never started

#### Pattern:
```java
exec.shutdown();                    // Initiate shutdown
exec.awaitTermination(timeout);     // Wait for completion
// Or combine for synchronous shutdown effect
```

---

#### Example: LifecycleWebServer

```java
class LifecycleWebServer {
    private final ExecutorService exec = ...;
    
    public void start() throws IOException {
        ServerSocket socket = new ServerSocket(80);
        while (!exec.isShutdown()) {
            try {
                final Socket conn = socket.accept();
                exec.execute(() -> handleRequest(conn));
            } catch (RejectedExecutionException e) {
                if (!exec.isShutdown())
                    log("task submission rejected", e);
            }
        }
    }
    
    public void stop() { exec.shutdown(); }
    
    void handleRequest(Socket connection) {
        Request req = readRequest(connection);
        if (isShutdownRequest(req))
            stop();  // Programmatic shutdown
        else
            dispatchRequest(req);
    }
}
```

**Features**:
- Programmatic shutdown via `stop()`
- Client-triggered shutdown via special HTTP request
- Proper handling of `RejectedExecutionException`

---

### 6.2.5 Delayed and Periodic Tasks

#### Timer Problems:
- ✗ **Single thread**: One long task delays all others
- ✗ **Unchecked exceptions**: Kill timer thread, no recovery ("thread leakage")
- ✗ **Confusing behavior**: Throws `IllegalStateException` for future submissions

```java
// Timer misbehavior example
public class OutOfTime {
    public static void main(String[] args) throws Exception {
        Timer timer = new Timer();
        timer.schedule(new ThrowTask(), 1);
        SECONDS.sleep(1);
        timer.schedule(new ThrowTask(), 1);  // IllegalStateException!
        SECONDS.sleep(5);
    }
    
    static class ThrowTask extends TimerTask {
        public void run() { throw new RuntimeException(); }
    }
}
```
**Expected**: Run for 6 seconds
**Actual**: Terminates after 1 second with `IllegalStateException`

#### ScheduledThreadPoolExecutor (Use Instead):
- ✓ Multiple threads for scheduled tasks
- ✓ Handles ill-behaved tasks properly
- ✓ More robust and flexible

**Building custom schedulers**: Use `DelayQueue` (implements `BlockingQueue` with delay semantics)

---

## 6.3 Finding Exploitable Parallelism

**Goal**: Identify good task boundaries to maximize concurrency

**Example domain**: HTML page rendering
- Parse HTML
- Render text elements
- Download images
- Display images

---

### 6.3.1 Sequential Page Renderer

```java
public class SingleThreadRenderer {
    void renderPage(CharSequence source) {
        renderText(source);  // Render text first
        List<ImageData> imageData = new ArrayList<>();
        for (ImageInfo imageInfo : scanForImageInfo(source))
            imageData.add(imageInfo.downloadImage());  // Download images
        for (ImageData data : imageData)
            renderImage(data);  // Display images
    }
}
```

**Problems**:
- ✗ Downloads images sequentially (wastes time)
- ✗ CPU idle during I/O waits
- ✗ User waits longer than necessary

---

### 6.3.2 Result-Bearing Tasks: Callable and Future

#### Limitations of Runnable:
- Cannot return a value
- Cannot throw checked exceptions
- Must use side effects (shared data structures, logs)

#### Callable - Better Abstraction:

```java
public interface Callable<V> {
    V call() throws Exception;  // Can return value and throw exceptions
}
```

#### Future - Task Lifecycle:

```java
public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException, CancellationException;
    V get(long timeout, TimeUnit unit) throws ...;
}
```

**Task states**: Created → Submitted → Started → Completed
- Lifecycle only moves forward
- Once completed, stays completed forever

**`get()` behavior**:
- Returns immediately if task completed
- Blocks if task still running
- Throws `ExecutionException` if task threw exception
- Throws `CancellationException` if task was cancelled

**Creating Futures**:
- `ExecutorService.submit(Callable)` returns `Future`
- `ExecutorService.submit(Runnable)` returns `Future<?>`
- Explicitly instantiate `FutureTask`

---

### 6.3.3 Example: Page Renderer with Future

```java
public class FutureRenderer {
    private final ExecutorService executor = ...;
    
    void renderPage(CharSequence source) {
        final List<ImageInfo> imageInfos = scanForImageInfo(source);
        
        Callable<List<ImageData>> task = () -> {
            List<ImageData> result = new ArrayList<>();
            for (ImageInfo imageInfo : imageInfos)
                result.add(imageInfo.downloadImage());
            return result;
        };
        
        Future<List<ImageData>> future = executor.submit(task);
        
        renderText(source);  // Render text while images download
        
        try {
            List<ImageData> imageData = future.get();  // Wait for images
            for (ImageData data : imageData)
                renderImage(data);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            future.cancel(true);
        } catch (ExecutionException e) {
            throw launderThrowable(e.getCause());
        }
    }
}
```

**Improvements**:
- ✓ Text rendering concurrent with image downloading
- ✓ User sees text quickly
- ✓ Better CPU utilization

**Limitation**: Still waits for ALL images before displaying any

---

### 6.3.4 Limitations of Parallelizing Heterogeneous Tasks

**Problem with heterogeneous parallelization**:

**Analogy**: Washing dishes
- Two people: One washes, one dries → works well
- More people: Hard to find roles, diminishing returns

**Issues**:
1. **Limited scalability**: Two tasks → max 2x speedup
2. **Disparate task sizes**: If one task is 10x longer, overall speedup < 2x
3. **Coordination overhead**: Must be outweighed by parallelism benefits

**Real performance payoff**: Large number of **independent, homogeneous** tasks

---

### 6.3.5 CompletionService: Executor Meets BlockingQueue

**Problem**: Retrieving results as they complete is tedious with `Future.get(0)`

**Solution**: `CompletionService`

```java
public interface CompletionService<V> {
    Future<V> submit(Callable<V> task);
    Future<V> submit(Runnable task, V result);
    Future<V> take() throws InterruptedException;      // Blocks until result available
    Future<V> poll();                                   // Returns null if none available
    Future<V> poll(long timeout, TimeUnit unit);       // Timed poll
}
```

**How it works** (`ExecutorCompletionService`):
- Wraps submitted tasks in `QueueingFuture`
- `QueueingFuture.done()` puts completed `Future` on `BlockingQueue`
- `take()`/`poll()` retrieve results from queue as they complete

---

### 6.3.6 Example: Page Renderer with CompletionService

```java
public class Renderer {
    private final ExecutorService executor;
    
    void renderPage(CharSequence source) {
        List<ImageInfo> info = scanForImageInfo(source);
        CompletionService<ImageData> completionService =
            new ExecutorCompletionService<>(executor);
        
        // Submit all download tasks
        for (final ImageInfo imageInfo : info)
            completionService.submit(() -> imageInfo.downloadImage());
        
        renderText(source);  // Render text while images download
        
        try {
            for (int t = 0, n = info.size(); t < n; t++) {
                Future<ImageData> f = completionService.take();  // Get next completed
                ImageData imageData = f.get();
                renderImage(imageData);  // Display as soon as ready
            }
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        } catch (ExecutionException e) {
            throw launderThrowable(e.getCause());
        }
    }
}
```

**Improvements**:
- ✓ **Parallel downloads**: All images download concurrently
- ✓ **Progressive rendering**: Display each image as it arrives
- ✓ **Better responsiveness**: User sees dynamic updates

**Pattern**: Track number of submitted tasks, count completions to know when batch is done

---

### 6.3.7 Placing Time Limits on Tasks

**Scenarios requiring timeouts**:
- Web ads that shouldn't delay page load
- Portal fetching data from multiple sources with time budget
- Any task where old results are worthless

**Primary challenge**: Don't wait longer than time budget

**Solution**: Timed `Future.get()`

```java
Page renderPageWithAd() throws InterruptedException {
    long endNanos = System.nanoTime() + TIME_BUDGET;
    Future<Ad> f = exec.submit(new FetchAdTask());
    
    Page page = renderPageBody();  // Render while ad loads
    Ad ad;
    
    try {
        long timeLeft = endNanos - System.nanoTime();
        ad = f.get(timeLeft, NANOSECONDS);  // Wait only remaining time
    } catch (ExecutionException e) {
        ad = DEFAULT_AD;
    } catch (TimeoutException e) {
        ad = DEFAULT_AD;
        f.cancel(true);  // Cancel task to free resources
    }
    
    page.setAd(ad);
    return page;
}
```

**Secondary challenge**: Stop tasks when time expires
- Avoids wasting resources on unused results
- Cancel via `Future.cancel(true)` if task is cancellable

---

### 6.3.8 Example: Travel Reservations Portal

**Scenario**: Fetch quotes from multiple travel companies with time budget

**Solution**: `invokeAll()` with timeout

```java
public List<TravelQuote> getRankedTravelQuotes(
        TravelInfo travelInfo, 
        Set<TravelCompany> companies,
        Comparator<TravelQuote> ranking, 
        long time, TimeUnit unit)
        throws InterruptedException {
    
    List<QuoteTask> tasks = new ArrayList<>();
    for (TravelCompany company : companies)
        tasks.add(new QuoteTask(company, travelInfo));
    
    // Submit all, wait for timeout
    List<Future<TravelQuote>> futures = exec.invokeAll(tasks, time, unit);
    
    List<TravelQuote> quotes = new ArrayList<>(tasks.size());
    Iterator<QuoteTask> taskIter = tasks.iterator();
    
    for (Future<TravelQuote> f : futures) {
        QuoteTask task = taskIter.next();
        try {
            quotes.add(f.get());  // Completed or cancelled
        } catch (ExecutionException e) {
            quotes.add(task.getFailureQuote(e.getCause()));
        } catch (CancellationException e) {
            quotes.add(task.getTimeoutQuote(e));  // Timed out
        }
    }
    
    Collections.sort(quotes, ranking);
    return quotes;
}
```

**`invokeAll()` behavior**:
- Takes collection of tasks, returns collection of `Future`s
- Maintains order (task collection iterator order)
- Returns when: all complete, thread interrupted, OR timeout expires
- Incomplete tasks are cancelled on timeout

---

## Summary

### Key Concepts:

1. **Task-based structure** simplifies development and enables concurrency
2. **Executor framework** decouples submission from execution
3. **Thread pools** provide resource management and better performance
4. **Sensible task boundaries** are crucial for exploiting parallelism

### Best Practices:

✓ **Use Executor instead of `new Thread()`**
- More flexible, configurable, manageable

✓ **Choose appropriate execution policy**
- Fixed pool for bounded resources
- Cached pool for many short tasks
- Single thread for sequential execution
- Scheduled pool for periodic tasks

✓ **Identify homogeneous, independent tasks**
- Real parallelism comes from many similar tasks
- Heterogeneous tasks have limited speedup

✓ **Use Future and Callable for results**
- Better than Runnable for deferred computations
- Provides cancellation and exception handling

✓ **Use CompletionService for progressive results**
- Process results as they become available
- Better responsiveness

✓ **Set time budgets where appropriate**
- Timed `get()` prevents indefinite waiting
- Cancel tasks that exceed budget

✓ **Properly handle ExecutorService lifecycle**
- Call `shutdown()` when done
- Handle `RejectedExecutionException`

### Anti-Patterns to Avoid:

✗ **Unbounded thread creation** (`new Thread()` per task)
✗ **Using Timer** (use `ScheduledThreadPoolExecutor` instead)
✗ **Ignoring InterruptedException** (propagate or restore status)
✗ **Not shutting down executors** (prevents JVM exit)

### Quick Reference:

| Need | Use |
|------|-----|
| Execute tasks with bounded threads | `newFixedThreadPool(n)` |
| Execute many short-lived tasks | `newCachedThreadPool()` |
| Sequential task execution | `newSingleThreadExecutor()` |
| Scheduled/periodic tasks | `newScheduledThreadPool(n)` |
| Get result from task | `Callable` + `Future` |
| Process results as they complete | `CompletionService` |
| Time-limited task execution | `Future.get(timeout, unit)` |
| Batch of tasks with timeout | `invokeAll(tasks, timeout, unit)` |

### Performance Tips:

- Size thread pools based on available CPUs and task characteristics (more in Chapter 8)
- Use `CompletionService` for large batches of independent tasks
- Cancel futures when results no longer needed
- Consider parallelizing only when tasks are numerous and independent

---

# Chapter 7: Cancellation and Shutdown - Summary

## Overview
Stopping tasks and threads safely, quickly, and reliably is one of the most challenging aspects of concurrent programming. Java provides no mechanism to forcibly stop threads; instead, it uses **interruption**—a cooperative mechanism where one thread asks another to stop. Proper cancellation and shutdown handling distinguishes well-behaved applications from those that merely work.

---

## 7.1 Task Cancellation

### Reasons for Cancellation:

1. **User-requested cancellation**: User clicks "Cancel" button in GUI or JMX interface
2. **Time-limited activities**: Search for best solution within time budget, cancel remaining tasks when time expires
3. **Application events**: One task finds solution, cancel all other search tasks
4. **Errors**: Disk full during web crawling → cancel other crawlers
5. **Shutdown**: Application/service shutdown requires handling in-progress and queued work

---

### Basic Cancellation with Volatile Flag

```java
@ThreadSafe
public class PrimeGenerator implements Runnable {
    @GuardedBy("this")
    private final List<BigInteger> primes = new ArrayList<>();
    private volatile boolean cancelled;  // Cancellation flag
    
    public void run() {
        BigInteger p = BigInteger.ONE;
        while (!cancelled) {  // Check flag periodically
            p = p.nextProbablePrime();
            synchronized (this) {
                primes.add(p);
            }
        }
    }
    
    public void cancel() { cancelled = true; }
    
    public synchronized List<BigInteger> get() {
        return new ArrayList<>(primes);
    }
}

// Usage: Let it run for one second
List<BigInteger> aSecondOfPrimes() throws InterruptedException {
    PrimeGenerator generator = new PrimeGenerator();
    new Thread(generator).start();
    try {
        SECONDS.sleep(1);
    } finally {
        generator.cancel();  // Ensure cancellation even if sleep interrupted
    }
    return generator.get();
}
```

**Cancellation Policy**: Specifies "how", "when", and "what" of cancellation
- **How**: Other code requests cancellation
- **When**: Task checks for cancellation
- **What**: Actions taken in response

**Problem**: Won't work if task is blocked (e.g., in `BlockingQueue.put()`)

---

### 7.1.1 Interruption

#### Why Interruption is Needed:

❌ **Broken approach** - flag won't be checked if blocked:
```java
class BrokenPrimeProducer extends Thread {
    private final BlockingQueue<BigInteger> queue;
    private volatile boolean cancelled = false;
    
    public void run() {
        try {
            BigInteger p = BigInteger.ONE;
            while (!cancelled)
                queue.put(p = p.nextProbablePrime());  // May block forever!
        } catch (InterruptedException consumed) { }
    }
    
    public void cancel() { cancelled = true; }  // Won't help if stuck in put()
}
```

#### Interruption API:

```java
public class Thread {
    public void interrupt() { ... }              // Set interrupted status
    public boolean isInterrupted() { ... }       // Query status
    public static boolean interrupted() { ... }  // Clear and return previous status
}
```

**Key behaviors**:
- Calling `interrupt()` sets the thread's interrupted status to `true`
- Blocking methods (`sleep`, `wait`, `join`, `put`, `take`) detect interruption and:
  - Clear interrupted status
  - Throw `InterruptedException`
  - Return early from blocking operation
- If thread not blocked when interrupted, status is set ("sticky") until explicitly cleared

**Interruption is usually the most sensible way to implement cancellation.**

---

#### Fixed Version Using Interruption:

```java
class PrimeProducer extends Thread {
    private final BlockingQueue<BigInteger> queue;
    
    public void run() {
        try {
            BigInteger p = BigInteger.ONE;
            while (!Thread.currentThread().isInterrupted())  // Check status
                queue.put(p = p.nextProbablePrime());  // Throws InterruptedException
        } catch (InterruptedException consumed) {
            /* Allow thread to exit */
        }
    }
    
    public void cancel() { interrupt(); }  // Use standard mechanism
}
```

**Two detection points**:
1. Explicit check in loop header (better responsiveness)
2. Implicit check in blocking `put()` call

---

### 7.1.2 Interruption Policies

**Interruption policy**: How a thread interprets interruption
- What to do when interruption detected
- What units of work are atomic with respect to interruption
- How quickly to react

**Most sensible policy**: Thread-level or service-level cancellation
- Exit as quickly as practical
- Clean up if necessary
- Notify owning entity

**Critical distinction**: Task vs. Thread interruption policy
- Single interrupt may have multiple recipients
- E.g., interrupting thread pool worker means both:
  - "Cancel current task"
  - "Shut down worker thread"

**Key principle**: **Tasks don't own threads**
- Tasks borrow threads from services (like thread pools)
- Code not owning thread should preserve interrupted status
- Let owning code eventually act on interruption

**Why blocking methods throw InterruptedException**:
- They don't execute in threads they own
- Most reasonable policy: Get out of the way quickly
- Communicate interruption back to caller

**You should not interrupt a thread unless you know what interruption means to that thread.**

---

### 7.1.3 Responding to Interruption

**Two proper strategies**:

#### 1. Propagate InterruptedException

```java
// Simplest approach - just add to throws clause
public Task getNextTask() throws InterruptedException {
    return queue.take();
}
```

#### 2. Restore Interrupted Status

```java
// When can't throw (e.g., Runnable)
public void run() {
    try {
        processTask(queue.take());
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();  // Restore status
    }
}
```

**Never do this**: ❌ Catch and swallow without restoring
```java
// WRONG - loses interruption information
catch (InterruptedException e) { 
    // Do nothing - BAD!
}
```

---

#### Non-Cancellable Activities

For code that **must complete** despite interruption:

```java
public Task getNextTask(BlockingQueue<Task> queue) {
    boolean interrupted = false;
    try {
        while (true) {
            try {
                return queue.take();
            } catch (InterruptedException e) {
                interrupted = true;  // Remember we were interrupted
                // Fall through and retry
            }
        }
    } finally {
        if (interrupted)
            Thread.currentThread().interrupt();  // Restore before returning
    }
}
```

**When to restore**:
- ✓ Just before returning (not immediately)
- ✗ Early restoration → infinite loop (interruptible methods check on entry)

**Only code implementing thread's interruption policy may swallow interruption.**

---

### 7.1.4 Example: Timed Run

**Goal**: Run task for specified time, then cancel

#### Attempt 1: Interrupt Borrowed Thread ❌

```java
// DON'T DO THIS - violates interruption policy
private static final ScheduledExecutorService cancelExec = ...;

public static void timedRun(Runnable r, long timeout, TimeUnit unit) {
    final Thread taskThread = Thread.currentThread();
    cancelExec.schedule(() -> taskThread.interrupt(), timeout, unit);
    r.run();
}
```

**Problems**:
1. Violates rule: Don't interrupt thread unless you know its policy
2. If task completes before timeout, interrupt fires after `timedRun` returns
3. If task not responsive to interrupt, `timedRun` doesn't return until task finishes

---

#### Attempt 2: Run in Dedicated Thread

```java
public static void timedRun(final Runnable r, long timeout, TimeUnit unit)
        throws InterruptedException {
    class RethrowableTask implements Runnable {
        private volatile Throwable t;
        public void run() {
            try { r.run(); }
            catch (Throwable t) { this.t = t; }
        }
        void rethrow() {
            if (t != null) throw launderThrowable(t);
        }
    }
    
    RethrowableTask task = new RethrowableTask();
    final Thread taskThread = new Thread(task);
    taskThread.start();
    cancelExec.schedule(() -> taskThread.interrupt(), timeout, unit);
    taskThread.join(unit.toMillis(timeout));
    task.rethrow();
}
```

**Problems**:
- Uses timed `join()` - can't tell if thread exited or join timed out
- More complex than necessary

---

### 7.1.5 Cancellation Via Future ✓

**Best approach**: Use `Future` and task execution framework

```java
public static void timedRun(Runnable r, long timeout, TimeUnit unit)
        throws InterruptedException {
    Future<?> task = taskExec.submit(r);
    try {
        task.get(timeout, unit);
    } catch (TimeoutException e) {
        // Task will be cancelled below
    } catch (ExecutionException e) {
        // Exception thrown in task; rethrow
        throw launderThrowable(e.getCause());
    } finally {
        // Harmless if task already completed
        task.cancel(true);  // Interrupt if running
    }
}
```

**`Future.cancel(mayInterruptIfRunning)`**:
- `true`: Interrupt thread if task currently running
- `false`: "Don't run if not started yet"
- Returns whether cancellation attempt successful

**When is `cancel(true)` safe?**
- Task execution threads from standard `Executor` implementations have interruption policy
- Safe to cancel tasks through `Future` when running in standard `Executor`

**When to cancel tasks**: When result no longer needed
- Shown in listings 6.13, 6.16, and 7.10

---

### 7.1.6 Dealing with Non-Interruptible Blocking

Some blocking operations **don't respond to interruption**:

#### 1. Synchronous Socket I/O (java.io)
**Problem**: `InputStream.read()` / `OutputStream.write()` ignore interruption

**Solution**: Close underlying socket
```java
public class ReaderThread extends Thread {
    private final Socket socket;
    private final InputStream in;
    
    public void interrupt() {
        try {
            socket.close();  // Causes SocketException in blocked read/write
        } catch (IOException ignored) { }
        finally {
            super.interrupt();
        }
    }
    
    public void run() {
        try {
            byte[] buf = new byte[BUFSZ];
            while (true) {
                int count = in.read(buf);
                if (count < 0) break;
                else if (count > 0) processBuffer(buf, count);
            }
        } catch (IOException e) { /* Allow thread to exit */ }
    }
}
```

#### 2. Synchronous I/O (java.nio)
**Behavior**: Interrupting thread waiting on `InterruptibleChannel`:
- Throws `ClosedByInterruptException`
- Closes the channel
- Other threads on same channel throw `ClosedByInterruptException`

**Closing `InterruptibleChannel`**: Throws `AsynchronousCloseException`

#### 3. Asynchronous I/O with Selector
**Behavior**: `Selector.select()` blocked → `wakeup()` returns prematurely
- Throws `ClosedSelectorException`

#### 4. Lock Acquisition
**Problem**: Waiting for intrinsic lock → interruption has no effect (except setting status)

**Solution**: Explicit locks with `lockInterruptibly()`
```java
Lock lock = new ReentrantLock();
try {
    lock.lockInterruptibly();  // Responsive to interruption
    // Use shared resource
} catch (InterruptedException e) {
    // Cleanup and exit
} finally {
    lock.unlock();
}
```

---

### 7.1.7 Encapsulating Nonstandard Cancellation with newTaskFor

**Java 6+**: Override `ThreadPoolExecutor.newTaskFor()` to customize cancellation

```java
public interface CancellableTask<T> extends Callable<T> {
    void cancel();
    RunnableFuture<T> newTask();
}

public class CancellingExecutor extends ThreadPoolExecutor {
    protected <T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
        if (callable instanceof CancellableTask)
            return ((CancellableTask<T>) callable).newTask();
        else
            return super.newTaskFor(callable);
    }
}

public abstract class SocketUsingTask<T> implements CancellableTask<T> {
    @GuardedBy("this") private Socket socket;
    
    protected synchronized void setSocket(Socket s) { socket = s; }
    
    public synchronized void cancel() {
        try {
            if (socket != null) socket.close();
        } catch (IOException ignored) { }
    }
    
    public RunnableFuture<T> newTask() {
        return new FutureTask<T>(this) {
            public boolean cancel(boolean mayInterruptIfRunning) {
                try {
                    SocketUsingTask.this.cancel();
                } finally {
                    return super.cancel(mayInterruptIfRunning);
                }
            }
        };
    }
}
```

**Benefits**: Custom cancellation can:
- Log cancellations
- Gather statistics
- Cancel non-interruptible activities

---

## 7.2 Stopping a Thread-Based Service

### Principles:

**Thread ownership**:
- Thread is owned by class that created it
- Thread pool owns worker threads
- Application owns services, services own threads
- **Ownership is not transitive**: App shouldn't stop worker threads directly

**Proper approach**: Service provides lifecycle methods

**Provide lifecycle methods whenever thread-owning service has lifetime longer than creating method.**

---

### 7.2.1 Example: A Logging Service

#### Simple Approach (No Shutdown Support):

```java
public class LogWriter {
    private final BlockingQueue<String> queue;
    private final LoggerThread logger;
    
    public LogWriter(Writer writer) {
        this.queue = new LinkedBlockingQueue<>(CAPACITY);
        this.logger = new LoggerThread(writer);
    }
    
    public void start() { logger.start(); }
    
    public void log(String msg) throws InterruptedException {
        queue.put(msg);
    }
    
    private class LoggerThread extends Thread {
        private final PrintWriter writer;
        
        public void run() {
            try {
                while (true)
                    writer.println(queue.take());
            } catch (InterruptedException ignored) {
            } finally {
                writer.close();
            }
        }
    }
}
```

---

#### Broken Shutdown Attempt ❌

```java
// DON'T DO THIS - race condition
public void log(String msg) throws InterruptedException {
    if (!shutdownRequested)  // Check
        queue.put(msg);      // Then act - RACE!
    else
        throw new IllegalStateException("logger is shut down");
}
```

**Problem**: Check-then-act race condition
- Producer sees not shut down, but shutdown occurs before put
- Producer might block forever in put

---

#### Correct Approach: Atomic Reservation ✓

```java
public class LogService {
    private final BlockingQueue<String> queue;
    private final LoggerThread loggerThread;
    private final PrintWriter writer;
    @GuardedBy("this") private boolean isShutdown;
    @GuardedBy("this") private int reservations;  // Messages reserved
    
    public void stop() {
        synchronized (this) { isShutdown = true; }
        loggerThread.interrupt();
    }
    
    public void log(String msg) throws InterruptedException {
        synchronized (this) {
            if (isShutdown)
                throw new IllegalStateException(...);
            ++reservations;  // Atomically reserve right to submit
        }
        queue.put(msg);
    }
    
    private class LoggerThread extends Thread {
        public void run() {
            try {
                while (true) {
                    try {
                        synchronized (LogService.this) {
                            if (isShutdown && reservations == 0)
                                break;  // All reserved messages processed
                        }
                        String msg = queue.take();
                        synchronized (LogService.this) { --reservations; }
                        writer.println(msg);
                    } catch (InterruptedException e) { /* retry */ }
                }
            } finally {
                writer.close();
            }
        }
    }
}
```

**How it works**:
1. Atomically check shutdown status and increment reservation counter
2. Logger processes messages until shutdown AND all reservations consumed
3. No messages lost, no producers stuck

---

### 7.2.2 ExecutorService Shutdown

**Using ExecutorService simplifies lifecycle management**:

```java
public class LogService {
    private final ExecutorService exec = newSingleThreadExecutor();
    private final PrintWriter writer;
    
    public void stop() throws InterruptedException {
        try {
            exec.shutdown();
            exec.awaitTermination(TIMEOUT, UNIT);
        } finally {
            writer.close();
        }
    }
    
    public void log(String msg) {
        try {
            exec.execute(new WriteTask(msg));
        } catch (RejectedExecutionException ignored) { }
    }
}
```

**Shutdown options**:
- `shutdown()`: Graceful - complete queued tasks
- `shutdownNow()`: Abrupt - cancel active tasks, return queued tasks

**Encapsulation**: Wrapping `ExecutorService` extends ownership chain
- Application → Service → ExecutorService → Threads

---

### 7.2.3 Poison Pills

**Poison pill**: Recognizable object meaning "when you get this, stop"

```java
public class IndexingService {
    private static final File POISON = new File("");  // Poison pill
    private final IndexerThread consumer = new IndexerThread();
    private final CrawlerThread producer = new CrawlerThread();
    private final BlockingQueue<File> queue;
    
    public void start() {
        producer.start();
        consumer.start();
    }
    
    public void stop() { producer.interrupt(); }
    
    public void awaitTermination() throws InterruptedException {
        consumer.join();
    }
    
    class CrawlerThread extends Thread {
        public void run() {
            try {
                crawl(root);
            } finally {
                while (true) {
                    try {
                        queue.put(POISON);  // Submit poison pill
                        break;
                    } catch (InterruptedException e) { /* retry */ }
                }
            }
        }
    }
    
    class IndexerThread extends Thread {
        public void run() {
            try {
                while (true) {
                    File file = queue.take();
                    if (file == POISON)  // Detect poison pill
                        break;
                    else
                        indexFile(file);
                }
            } catch (InterruptedException consumed) { }
        }
    }
}
```

**Limitations**:
- ✓ Works with FIFO queues (ensures prior work completed)
- ✓ Simple for known producer/consumer counts
- ✗ Multiple producers: Each puts one pill, consumer stops after N pills
- ✗ Multiple consumers: Producer puts N pills
- ✗ Unbounded queues only (bounded might block producer trying to submit pill)

---

### 7.2.4 Example: One-Shot Execution Service

**Private executor for method-scoped batch processing**:

```java
boolean checkMail(Set<String> hosts, long timeout, TimeUnit unit)
        throws InterruptedException {
    ExecutorService exec = Executors.newCachedThreadPool();
    final AtomicBoolean hasNewMail = new AtomicBoolean(false);
    
    try {
        for (final String host : hosts)
            exec.execute(() -> {
                if (checkMail(host))
                    hasNewMail.set(true);
            });
    } finally {
        exec.shutdown();
        exec.awaitTermination(timeout, unit);
    }
    return hasNewMail.get();
}
```

**Pattern**: Executor lifetime bounded by method
- Simplifies lifecycle management
- `invokeAll()` often useful for this pattern

---

### 7.2.5 Limitations of shutdownNow

**Problem**: `shutdownNow()` returns tasks that never started, but not tasks in progress

**Solution**: Track in-progress tasks

```java
public class TrackingExecutor extends AbstractExecutorService {
    private final ExecutorService exec;
    private final Set<Runnable> tasksCancelledAtShutdown =
        Collections.synchronizedSet(new HashSet<>());
    
    public List<Runnable> getCancelledTasks() {
        if (!exec.isTerminated())
            throw new IllegalStateException(...);
        return new ArrayList<>(tasksCancelledAtShutdown);
    }
    
    public void execute(final Runnable runnable) {
        exec.execute(new Runnable() {
            public void run() {
                try {
                    runnable.run();
                } finally {
                    if (isShutdown() && Thread.currentThread().isInterrupted())
                        tasksCancelledAtShutdown.add(runnable);
                }
            }
        });
    }
}
```

**How it works**:
- Wraps tasks to remember if cancelled after shutdown
- Task must preserve interrupted status for this to work
- Returns cancelled tasks after termination

---

#### Example: Web Crawler with State Preservation

```java
public abstract class WebCrawler {
    private volatile TrackingExecutor exec;
    @GuardedBy("this")
    private final Set<URL> urlsToCrawl = new HashSet<>();
    
    public synchronized void start() {
        exec = new TrackingExecutor(Executors.newCachedThreadPool());
        for (URL url : urlsToCrawl) submitCrawlTask(url);
        urlsToCrawl.clear();
    }
    
    public synchronized void stop() throws InterruptedException {
        try {
            saveUncrawled(exec.shutdownNow());  // Tasks never started
            if (exec.awaitTermination(TIMEOUT, UNIT))
                saveUncrawled(exec.getCancelledTasks());  // Tasks in progress
        } finally {
            exec = null;
        }
    }
    
    private void saveUncrawled(List<Runnable> uncrawled) {
        for (Runnable task : uncrawled)
            urlsToCrawl.add(((CrawlTask) task).getPage());
    }
    
    private class CrawlTask implements Runnable {
        private final URL url;
        
        public void run() {
            for (URL link : processPage(url)) {
                if (Thread.currentThread().isInterrupted())
                    return;  // Preserve interrupted status
                submitCrawlTask(link);
            }
        }
        
        public URL getPage() { return url; }
    }
}
```

**Caveat**: Race condition → false positives
- Task might complete between last instruction and pool recording completion
- Safe if tasks are **idempotent** (performing twice = same as once)
- Otherwise, app must handle false positives

---

## 7.3 Handling Abnormal Thread Termination

### The Problem:

**Uncaught exceptions** can kill threads:
- Single-threaded console app: Obvious failure (stack trace, program stops)
- Concurrent app: Thread dies silently, app appears to continue
- Thread pool: Lost worker thread → reduced capacity
- GUI event thread: Application freezes
- Timer thread: Service permanently out of commission

**Leading cause**: `RuntimeException` (programming errors, unrecoverable problems)

---

### Prevention Strategy:

**Worker thread structure**:

```java
public void run() {
    Throwable thrown = null;
    try {
        while (!isInterrupted())
            runTask(getTaskFromWorkQueue());
    } catch (Throwable e) {
        thrown = e;
    } finally {
        threadExited(this, thrown);  // Notify framework
    }
}
```

**Key pattern**: Catch `Throwable` in abstraction barriers
- Use when calling unknown/untrusted code
- Thread pools, Swing event dispatch use this technique
- Ensures one bad task doesn't kill the thread

---

### 7.3.1 Uncaught Exception Handlers

**API** (Java 5.0+):

```java
public interface UncaughtExceptionHandler {
    void uncaughtException(Thread t, Throwable e);
}
```

**Setting handlers**:
1. Per-thread: `thread.setUncaughtExceptionHandler(handler)`
2. Default: `Thread.setDefaultUncaughtExceptionHandler(handler)`

**Handler search order**:
1. Per-thread handler
2. ThreadGroup handler (delegates to parent)
3. Top-level ThreadGroup handler
4. Default system handler
5. Print to `System.err`

---

**Example handler**:

```java
public class UEHLogger implements Thread.UncaughtExceptionHandler {
    public void uncaughtException(Thread t, Throwable e) {
        Logger logger = Logger.getAnonymousLogger();
        logger.log(Level.SEVERE,
            "Thread terminated with exception: " + t.getName(), e);
    }
}
```

**Best practice**: Always use handlers in long-running applications to at least log exceptions

---

**Setting for thread pools**:

```java
ThreadFactory factory = r -> {
    Thread t = new Thread(r);
    t.setUncaughtExceptionHandler(new UEHLogger());
    return t;
};
ExecutorService exec = Executors.newCachedThreadPool(factory);
```

---

**Important caveat**: Handlers only invoked for tasks submitted with `execute()`

**For tasks submitted with `submit()`**:
- Any exception (checked or unchecked) is part of return status
- Exception wrapped in `ExecutionException`
- Rethrown by `Future.get()`

**Alternative**: Override `afterExecute()` hook in `ThreadPoolExecutor`

---

## 7.4 JVM Shutdown

### Shutdown Types:

#### Orderly Shutdown (Preferred):
- Last normal (non-daemon) thread terminates
- Someone calls `System.exit()`
- Platform-specific means (SIGINT, Ctrl-C)

#### Abrupt Shutdown:
- `Runtime.halt()` called
- JVM process killed (SIGKILL)

---

### 7.4.1 Shutdown Hooks

**Shutdown hooks**: Unstarted threads registered with `Runtime.addShutdownHook()`

**Orderly shutdown sequence**:
1. Start all registered shutdown hooks (concurrently, no ordering guaranteed)
2. Application threads continue concurrently with shutdown hooks
3. When all hooks complete, run finalizers if `runFinalizersOnExit` is true
4. Halt JVM

**Abrupt shutdown**: Shutdown hooks do NOT run

---

**Example: Logging service shutdown hook**:

```java
public void start() {
    Runtime.getRuntime().addShutdownHook(new Thread() {
        public void run() {
            try { 
                LogService.this.stop(); 
            } catch (InterruptedException ignored) {}
        }
    });
}
```

---

**Shutdown hook requirements**:

⚠️ **Must be**:
- **Thread-safe**: Use synchronization, avoid deadlock
- **Defensive**: Don't assume app state or why JVM is shutting down
- **Fast**: Exit quickly (user expects quick termination)

❌ **Don't**:
- Rely on services that might be shut down by app or other hooks
- Make assumptions about shutdown order

**Best practice**: Use single shutdown hook for all services
- Ensures sequential execution (avoids race conditions/deadlock)
- Controls shutdown order based on dependency information

---

### 7.4.2 Daemon Threads

**Daemon thread**: Helper thread that shouldn't prevent JVM shutdown

**Characteristics**:
- Created by JVM (GC, housekeeping) except main thread
- Inherit daemon status from creating thread
- Default: main thread is normal → all created threads are normal

**Behavior at shutdown**:
- JVM inventories running threads
- If only daemon threads remain → orderly shutdown
- Daemon threads are **abandoned** when JVM halts:
  - `finally` blocks NOT executed
  - Stacks NOT unwound
  - JVM just exits

---

**When to use**:
- ✓ Housekeeping tasks (periodic cache cleanup)
- ✗ I/O operations (can't be safely abandoned)
- ✗ Any task requiring cleanup

**Daemon threads are not a substitute for proper lifecycle management.**

---

### 7.4.3 Finalizers

**Purpose**: Release resources (file handles, sockets) when object garbage collected

**Problems**:
- ❌ No guarantee when (or if) finalizers run
- ❌ Significant performance cost
- ❌ Extremely difficult to write correctly
- ❌ Finalize runs in JVM-managed thread → requires synchronization

**Better alternatives**:
- `finally` blocks
- Explicit `close()` methods

**Exception**: Resources acquired by native methods

**Best practice**: Avoid finalizers (except platform library classes)

---

## Summary

### Core Principles:

1. **No preemptive cancellation** - Java provides cooperative interruption only
2. **Interruption is just a request** - thread chooses when to respond
3. **Preserve interrupted status** - if not handling, restore for higher-level code
4. **Interruption usually means cancellation** - use it consistently
5. **Own the thread to interrupt it** - only thread owner should set interruption policy
6. **Services need lifecycle methods** - provide shutdown mechanisms for thread-owning services
7. **Always use uncaught exception handlers** - at minimum, log the exception

---

### Cancellation Checklist:

#### Implementing Cancellable Tasks:

✓ Use interruption as cancellation mechanism
✓ Check interrupted status before expensive operations
✓ Call interruptible blocking methods when possible
✓ Preserve interrupted status if can't throw `InterruptedException`
✓ Clean up resources in `finally` blocks
✓ Document cancellation policy

#### Cancelling Tasks:

✓ Cancel through `Future.cancel()` when using `ExecutorService`
✓ Use `cancel(true)` to interrupt if task supports it
✓ Don't interrupt pool threads directly
✓ Know the interruption policy before interrupting
✓ Cancel tasks whose results no longer needed

---

### Responding to InterruptedException:

| Situation | Action |
|-----------|--------|
| Can propagate exception | Add `throws InterruptedException` to method signature |
| Can't propagate (Runnable) | Catch, restore status: `Thread.currentThread().interrupt()` |
| Implementing interruption policy | Can swallow (but restore status before exiting scope) |
| Non-cancellable task | Catch, save status, retry, restore in `finally` |

---

### Shutdown Patterns:

| Pattern | Use When | Example |
|---------|----------|---------|
| Poison pill | FIFO queue, known producer/consumer count | Desktop search indexer |
| Shutdown flag + reservation | Prevent lost messages | Logging service |
| ExecutorService delegation | Standard lifecycle needed | Logging with ExecutorService |
| Shutdown hook | JVM shutdown cleanup | Close log file on exit |
| TrackingExecutor | Need to know cancelled tasks | Web crawler state preservation |

---

### Thread Lifecycle:

```
Normal Thread:
  Created → Started → Running → Terminated
           ↓                ↓
         Blocked      Interrupted → Cleanup → Exit
         
Daemon Thread:
  Created → Started → Running → Abandoned (if JVM shuts down)
```

---

### Common Pitfalls:

❌ **Swallowing interrupts** without restoring status
❌ **Interrupting borrowed threads** (violates ownership)
❌ **Using volatile flag** for tasks that might block
❌ **Check-then-act** shutdown flag without atomicity
❌ **Not handling uncaught exceptions** in worker threads
❌ **Relying on finalizers** for cleanup
❌ **Using daemon threads** for I/O or tasks needing cleanup
❌ **Shutdown hooks** that depend on other services/hooks
❌ **Not tracking in-progress tasks** when using `shutdownNow()`

---

### Best Practices Summary:

**Task Development**:
- Make tasks responsive to interruption
- Use `Future` and `Callable` for cancellable, result-bearing tasks
- Document and follow consistent cancellation policy

**Service Development**:
- Provide `shutdown()` methods for thread-owning services
- Support both graceful and abrupt shutdown when appropriate
- Track in-progress work if state must be preserved

**Application Development**:
- Always shut down `ExecutorService` instances
- Use shutdown hooks for critical cleanup only
- Set uncaught exception handlers on all threads
- Test shutdown and cancellation paths thoroughly

**General**:
- Favor `ExecutorService` over manual thread management
- Use standard patterns (poison pill, shutdown methods)
- Understand and document interruption policies
- Handle abnormal termination gracefully

---

# Chapter 8: Applying Thread Pools

## Overview

Chapter 8 covers advanced configuration and tuning of thread pools, exploring implicit couplings between tasks and execution policies, sizing strategies, configuration options, and techniques for parallelizing recursive algorithms.

**Key Topics**:
- Implicit constraints between tasks and execution policies
- Thread pool sizing calculations
- Configuring `ThreadPoolExecutor` (queues, saturation policies, thread factories)
- Extending thread pools for custom behavior
- Parallelizing sequential and recursive algorithms
- Practical example: concurrent puzzle solver framework

---

## 8.1 Implicit Couplings Between Tasks and Execution Policies

### The Decoupling Myth

While the Executor framework provides flexibility, **not all tasks are compatible with all execution policies**.

---

### Task Types Requiring Specific Execution Policies:

#### 1. **Dependent Tasks**

**Independent tasks** (best behaved):
- Don't depend on timing, results, or side effects of other tasks
- Can freely vary pool size and configuration
- Only performance is affected

**Dependent tasks**:
- Create constraints on execution policy
- Risk liveness problems if not carefully managed
- May require specific pool sizes or unbounded queues

---

#### 2. **Tasks Exploiting Thread Confinement**

**Single-threaded executors** provide:
- Guarantee that tasks don't execute concurrently
- Allow relaxed thread safety in task code
- Enable object confinement to task thread

**Example**:
```java
// Safe ONLY with single-threaded executor
ExecutorService singleThreadExec = Executors.newSingleThreadExecutor();
singleThreadExec.execute(() -> {
    // Can safely access non-thread-safe objects
    nonThreadSafeObject.update();
});
```

⚠️ **Switching to thread pool breaks thread safety**

**Requirement**: Tasks must be single-threaded OR provide sufficient synchronization for memory visibility

---

#### 3. **Response-Time-Sensitive Tasks**

**Problem**: Long-running tasks block short tasks

**GUI applications**:
- Users expect immediate visual feedback
- Submitting long-running task to small pool impairs responsiveness

**Solutions**:
- Use separate thread pools for different task types
- Bound task execution time with timeouts
- Prioritize short tasks with `PriorityBlockingQueue`

---

#### 4. **Tasks Using ThreadLocal**

**ThreadLocal behavior**:
- Provides per-thread private variables
- Executors reuse threads arbitrarily
- Values may leak between unrelated tasks

⚠️ **ThreadLocal appropriate ONLY if value lifetime is bounded by task**

**Don't use ThreadLocal to communicate between pool tasks**

---

### Best Practice: Document Execution Policy Requirements

**Thread pools work best with**:
- **Homogeneous tasks** (similar duration/behavior)
- **Independent tasks** (no dependencies)

**Problems to avoid**:
- Mixing long/short tasks → clogging
- Dependent tasks → deadlock (unless pool unbounded)

📝 **Document requirements** so future maintainers don't break safety/liveness

---

## 8.1.1 Thread Starvation Deadlock

### Definition

**Thread starvation deadlock** occurs when a pool task waits for results from another task that's queued but can't execute due to pool size limits.

---

### Classic Example: Single-Threaded Executor

```java
// DON'T DO THIS - Always deadlocks!
public class ThreadDeadlock {
    ExecutorService exec = Executors.newSingleThreadExecutor();
    
    public class RenderPageTask implements Callable<String> {
        public String call() throws Exception {
            Future<String> header = exec.submit(new LoadFileTask("header.html"));
            Future<String> footer = exec.submit(new LoadFileTask("footer.html"));
            String page = renderBody();
            
            // DEADLOCK: Waiting for tasks that can't start
            return header.get() + page + footer.get();
        }
    }
}
```

**Why it deadlocks**:
1. `RenderPageTask` executes in the only thread
2. Submits `LoadFileTask` for header and footer
3. Waits for results via `get()`
4. But submitted tasks can't start (queue full, no threads)
5. **Circular wait**: main task waits for subtasks, subtasks wait for thread

---

### Larger Thread Pools

**Same problem** if:
- All threads execute tasks waiting on queued tasks
- Pool not large enough for dependency depth

**Example**: 10 threads, but tasks create 3-level dependency chains
- Top-level tasks: 10 (all threads busy)
- Second-level: 30 (queued)
- Third-level: 90 (queued)
- Deadlock when all 10 threads wait on level 2

---

### Mitigation Strategies:

#### 1. **Use Unbounded Thread Pools**
```java
ExecutorService exec = Executors.newCachedThreadPool();
```
- Creates threads as needed
- Avoids queueing when threads available
- Risk: unbounded resource consumption

#### 2. **Size Pool for Dependency Depth**
```java
// If tasks spawn N subtasks and wait
int poolSize = N + 1; // Or more for multiple concurrent parent tasks
```

#### 3. **Avoid Waiting on Subtask Results**
```java
// Instead of blocking on get()
Future<String> header = exec.submit(headerTask);
Future<String> footer = exec.submit(footerTask);

// Submit continuation task
exec.submit(() -> {
    String h = header.get();
    String f = footer.get();
    processResults(h, f);
});
```

#### 4. **Use Semaphore or Latch Instead**
```java
CountDownLatch latch = new CountDownLatch(2);
exec.submit(() -> { loadHeader(); latch.countDown(); });
exec.submit(() -> { loadFooter(); latch.countDown(); });
latch.await();
```

---

### Implicit Resource Constraints

**Hidden pool size limits**:
- JDBC connection pool with 10 connections
- Each task needs database connection
- **Effective thread pool size: 10** (others block waiting)

💡 **Identify all resource bottlenecks** when sizing pools

---

## 8.1.2 Long-Running Tasks

### Problems

**Thread pool clogged with long tasks**:
- Increases service time for short tasks
- Reduces responsiveness
- May not be deadlock, but still problematic

---

### Solution: Timed Resource Waits

**Use timed versions of blocking methods**:

```java
// Instead of unbounded wait
queue.put(item);  // Blocks forever if full

// Use timed wait
if (!queue.offer(item, 10, TimeUnit.SECONDS)) {
    // Mark task as failed, abort, or requeue
    handleTimeout();
}
```

**Platform methods with timed versions**:
- `Thread.join(timeout)`
- `BlockingQueue.poll(timeout, unit)`
- `CountDownLatch.await(timeout, unit)`
- `Selector.select(timeout)`
- `Future.get(timeout, unit)`

---

### Benefits

✅ **Guaranteed progress**: Each task eventually completes (success or failure)
✅ **Frees threads**: Don't block indefinitely
✅ **Diagnostic signal**: Frequent timeouts indicate pool too small

---

## 8.2 Sizing Thread Pools

### Guiding Principles

❌ **Don't hard-code pool sizes**
✅ **Use configuration or dynamic computation**

```java
int nCpus = Runtime.getRuntime().availableProcessors();
```

---

### Avoid Extremes

**Too large**:
- Threads compete for CPU and memory
- Higher memory usage
- More context switching
- Risk of resource exhaustion

**Too small**:
- Processors idle despite available work
- Throughput suffers

---

### Sizing Formula

**For compute-intensive tasks**:
```
N_threads = N_cpu + 1
```
- Extra thread prevents CPU idle during page faults

**For tasks with I/O or blocking**:
```
N_threads = N_cpu * U_cpu * (1 + W/C)

Where:
  N_cpu = number of CPUs
  U_cpu = target CPU utilization (0 ≤ U_cpu ≤ 1)
  W/C   = ratio of wait time to compute time
```

---

### Example Calculation

**Scenario**:
- 8-core system
- Target 80% CPU utilization
- Tasks spend 50% time waiting (W/C = 1)

```
N_threads = 8 * 0.8 * (1 + 1) = 12.8 ≈ 13 threads
```

---

### Estimating Wait/Compute Ratio

**Methods**:
1. **Profiling**: Use profiler to measure thread states
2. **Instrumentation**: Add timing code (see Section 8.4.1)
3. **Benchmarking**: Try different pool sizes, measure CPU utilization

---

### Resource-Based Constraints

**Other resources** limit pool size:
- **Memory**: Each thread has stack (typically 0.5-1 MB)
- **File handles**: OS limits per process
- **Socket handles**: Network connections
- **Database connections**: Connection pool size

**Formula for resource constraints**:
```
N_threads ≤ Total_resource / Resource_per_task
```

**Example**: 100 database connections, each task needs 2:
```
N_threads ≤ 100 / 2 = 50
```

---

### Multiple Thread Pools

**Consider separate pools** when:
- Tasks have very different behaviors (CPU vs I/O bound)
- Different priority requirements
- Different resource needs

**Example**:
```java
// Fast request handler
ExecutorService quickService = Executors.newFixedThreadPool(4);

// Heavy processing
ExecutorService heavyService = Executors.newFixedThreadPool(20);

// I/O intensive
ExecutorService ioService = Executors.newCachedThreadPool();
```

---

## 8.3 Configuring ThreadPoolExecutor

### General Constructor

```java
public ThreadPoolExecutor(
    int corePoolSize,              // Target size
    int maximumPoolSize,           // Max threads
    long keepAliveTime,            // Idle thread lifetime
    TimeUnit unit,                 // Time unit for keepAliveTime
    BlockingQueue<Runnable> workQueue,      // Task queue
    ThreadFactory threadFactory,   // Thread creation
    RejectedExecutionHandler handler        // Saturation policy
) { ... }
```

---

## 8.3.1 Thread Creation and Teardown

### Core Pool Size

**Target number of threads**:
- Pool maintains this size even when idle
- Won't create more unless work queue is full

⚠️ **Exception**: By default, core threads created on demand, not at construction

**Eager creation**:
```java
executor.prestartAllCoreThreads();
```

---

### Maximum Pool Size

**Upper bound** on active threads:
- Only relevant when using bounded queue
- With unbounded queue, never exceeds core size

---

### Keep-Alive Time

**Idle thread reaping**:
- Threads idle longer than keep-alive are terminated
- Only if current pool size > core size

**Java 6**: Allow core threads to time out:
```java
executor.allowCoreThreadTimeOut(true);
// Now ALL threads can be reaped when idle
```

---

### Factory Configurations

#### newFixedThreadPool
```java
new ThreadPoolExecutor(
    nThreads, nThreads,           // Core = Max
    0L, TimeUnit.MILLISECONDS,    // Infinite timeout
    new LinkedBlockingQueue<Runnable>()
)
```
- Fixed size pool
- Never times out
- Unbounded queue

#### newCachedThreadPool
```java
new ThreadPoolExecutor(
    0, Integer.MAX_VALUE,         // Expandable
    60L, TimeUnit.SECONDS,        // 1-minute timeout
    new SynchronousQueue<Runnable>()
)
```
- Infinitely expandable
- Threads reap after 1 minute idle
- No queuing (direct handoff)

---

## 8.3.2 Managing Queued Tasks

### Queue Types

#### 1. **Unbounded Queue** (LinkedBlockingQueue)

```java
new LinkedBlockingQueue<Runnable>()
```

**Characteristics**:
- Tasks queue up if all threads busy
- Queue can grow without bound
- Pool never exceeds core size

**Used by**: `newFixedThreadPool`, `newSingleThreadExecutor`

⚠️ **Risk**: Memory exhaustion if tasks arrive faster than processed

---

#### 2. **Bounded Queue**

```java
new LinkedBlockingQueue<Runnable>(capacity)
new ArrayBlockingQueue<Runnable>(capacity)
new PriorityBlockingQueue<Runnable>(capacity)
```

**Characteristics**:
- Prevents resource exhaustion
- Triggers saturation policy when full
- Must tune queue size with pool size

**Tradeoffs**:
- Large queue + small pool: Low resource usage, constrained throughput
- Small queue + large pool: Higher resource usage, better throughput

---

#### 3. **Synchronous Handoff** (SynchronousQueue)

```java
new SynchronousQueue<Runnable>()
```

**Characteristics**:
- Not really a queue (capacity 0)
- Direct handoff from producer to consumer
- If no thread waiting, creates new thread (up to max)
- If max reached, applies saturation policy

**Used by**: `newCachedThreadPool`

**Best for**: Unbounded pools or when rejecting tasks is acceptable

⚠️ **Only practical** if pool unbounded or rejection acceptable

---

#### 4. **Priority Queue**

```java
new PriorityBlockingQueue<Runnable>()
```

**Characteristics**:
- Tasks executed by priority, not arrival order
- Requires `Comparable` tasks or `Comparator`

**Example**:
```java
class PriorityTask implements Runnable, Comparable<PriorityTask> {
    private int priority;
    
    public int compareTo(PriorityTask other) {
        return Integer.compare(this.priority, other.priority);
    }
    
    public void run() { /* ... */ }
}
```

---

### Queue Selection Guidelines

| Pool Type | Best Queue | Reason |
|-----------|------------|--------|
| Unbounded pool | SynchronousQueue | Direct handoff, no queuing needed |
| Bounded pool (general) | Bounded queue | Prevents resource exhaustion |
| Bounded pool (independent tasks) | Unbounded queue | Simplicity, smooth bursts |
| Tasks with dependencies | Unbounded pool + queue | Avoid thread starvation deadlock |
| Priority-based | PriorityBlockingQueue | Control execution order |

**Default choice**: `newCachedThreadPool` (for most applications)
**Resource-constrained**: `newFixedThreadPool` with bounded queue

---

## 8.3.3 Saturation Policies

### When Saturation Occurs

**Bounded queue fills** and:
- Pool at maximum size
- Or executor shut down

---

### Built-in Policies

#### 1. **AbortPolicy** (Default)

```java
new ThreadPoolExecutor.AbortPolicy()
```

**Behavior**: Throws `RejectedExecutionException`

**Usage**:
```java
try {
    executor.execute(task);
} catch (RejectedExecutionException e) {
    // Handle overflow (log, queue elsewhere, etc.)
}
```

**Best for**: When caller should know about rejection

---

#### 2. **DiscardPolicy**

```java
new ThreadPoolExecutor.DiscardPolicy()
```

**Behavior**: Silently discards new task

**Best for**: Tasks that are optional or can be lost

---

#### 3. **DiscardOldestPolicy**

```java
new ThreadPoolExecutor.DiscardOldestPolicy()
```

**Behavior**:
- Discards task at head of queue (oldest/next to execute)
- Tries to resubmit new task

⚠️ **Don't combine** with `PriorityBlockingQueue` (discards highest priority!)

**Best for**: When newer tasks are more important

---

#### 4. **CallerRunsPolicy** (Throttling)

```java
new ThreadPoolExecutor.CallerRunsPolicy()
```

**Behavior**: Executes task in calling thread

**Example**:
```java
executor.setRejectedExecutionHandler(
    new ThreadPoolExecutor.CallerRunsPolicy()
);

// In main thread
executor.execute(task); // Runs in main thread if pool saturated
```

**Effect**:
- Slows down task submission
- Provides graceful degradation
- Pushes back-pressure to caller

**Degradation chain**:
```
Pool threads → Work queue → Application → TCP layer → Client
```

**Best for**: Web servers, request handlers (smooth degradation)

---

### Example: Fixed Pool with CallerRuns

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    N_THREADS, N_THREADS,
    0L, TimeUnit.MILLISECONDS,
    new LinkedBlockingQueue<Runnable>(CAPACITY)
);

executor.setRejectedExecutionHandler(
    new ThreadPoolExecutor.CallerRunsPolicy()
);
```

---

### Custom Saturation Policy: Blocking Submit

**Goal**: Block when queue full instead of rejecting

**Implementation using Semaphore**:

```java
@ThreadSafe
public class BoundedExecutor {
    private final Executor exec;
    private final Semaphore semaphore;
    
    public BoundedExecutor(Executor exec, int bound) {
        this.exec = exec;
        this.semaphore = new Semaphore(bound);
    }
    
    public void submitTask(final Runnable command) 
            throws InterruptedException {
        semaphore.acquire();  // Block if at capacity
        try {
            exec.execute(new Runnable() {
                public void run() {
                    try {
                        command.run();
                    } finally {
                        semaphore.release();
                    }
                }
            });
        } catch (RejectedExecutionException e) {
            semaphore.release();
            throw e;
        }
    }
}
```

**Bound semantics**: Limits tasks executing + queued

**Usage**:
```java
Executor exec = Executors.newCachedThreadPool();
BoundedExecutor bounded = new BoundedExecutor(exec, 100);

bounded.submitTask(task); // Blocks if 100 tasks in flight
```

---

## 8.3.4 Thread Factories

### ThreadFactory Interface

```java
public interface ThreadFactory {
    Thread newThread(Runnable r);
}
```

**Called** whenever pool needs to create thread

---

### Customization Reasons

1. **Naming threads** for easier debugging
2. **Setting UncaughtExceptionHandler**
3. **Custom Thread subclass**
4. **Modifying priority** (not recommended)
5. **Daemon status** (not recommended)
6. **Logging/monitoring** thread creation

---

### Example: Custom Thread Factory

```java
public class MyThreadFactory implements ThreadFactory {
    private final String poolName;
    
    public MyThreadFactory(String poolName) {
        this.poolName = poolName;
    }
    
    public Thread newThread(Runnable r) {
        return new MyAppThread(r, poolName);
    }
}
```

---

### Custom Thread Class with Debugging

```java
public class MyAppThread extends Thread {
    private static final String DEFAULT_NAME = "MyAppThread";
    private static volatile boolean debugLifecycle = false;
    private static final AtomicInteger created = new AtomicInteger();
    private static final AtomicInteger alive = new AtomicInteger();
    private static final Logger log = Logger.getAnonymousLogger();
    
    public MyAppThread(Runnable r, String poolName) {
        super(r, poolName + "-" + created.incrementAndGet());
        setUncaughtExceptionHandler(new Thread.UncaughtExceptionHandler() {
            public void uncaughtException(Thread t, Throwable e) {
                log.log(Level.SEVERE, 
                    "UNCAUGHT in thread " + t.getName(), e);
            }
        });
    }
    
    public void run() {
        boolean debug = debugLifecycle;
        if (debug) log.log(Level.FINE, "Created " + getName());
        try {
            alive.incrementAndGet();
            super.run();
        } finally {
            alive.decrementAndGet();
            if (debug) log.log(Level.FINE, "Exiting " + getName());
        }
    }
    
    public static int getThreadsCreated() { return created.get(); }
    public static int getThreadsAlive() { return alive.get(); }
    public static void setDebug(boolean b) { debugLifecycle = b; }
}
```

**Features**:
- Unique thread names with counter
- Uncaught exception handler
- Lifecycle logging (optional)
- Statistics (threads created/alive)

---

### Privileged Thread Factory

**For security-sensitive applications**:

```java
ExecutorService exec = Executors.newCachedThreadPool(
    Executors.privilegedThreadFactory()
);
```

**Creates threads with**:
- Same permissions as creating thread
- Same `AccessControlContext`
- Same `contextClassLoader`

**Without this**: Pool threads inherit permissions from submitting thread (can cause security issues)

---

## 8.3.5 Customizing After Construction

### Modifying via Setters

```java
ThreadPoolExecutor executor = 
    (ThreadPoolExecutor) Executors.newCachedThreadPool();

executor.setCorePoolSize(10);
executor.setMaximumPoolSize(20);
executor.setKeepAliveTime(30, TimeUnit.SECONDS);
executor.setThreadFactory(new MyThreadFactory("MyPool"));
executor.setRejectedExecutionHandler(new CallerRunsPolicy());
```

**Configurable properties**:
- Core pool size
- Maximum pool size
- Keep-alive time
- Thread factory
- Rejected execution handler

---

### Preventing Reconfiguration

**Problem**: Don't want clients changing execution policy

**Solution**: Wrap with `unconfigurableExecutorService`

```java
ExecutorService exec = Executors.unconfigurableExecutorService(
    Executors.newCachedThreadPool()
);

// exec only exposes ExecutorService methods
// Cannot cast to ThreadPoolExecutor
```

**Used by**: `newSingleThreadExecutor`

**Reason**: Preserve single-threaded semantics
- Prevent increasing pool size
- Maintain execution guarantees (no concurrent tasks)

---

## 8.4 Extending ThreadPoolExecutor

### Extension Hooks

Override these methods in subclass:

```java
protected void beforeExecute(Thread t, Runnable r) { }
protected void afterExecute(Runnable r, Throwable t) { }
protected void terminated() { }
```

---

### Hook Semantics

#### beforeExecute
- **Called in**: Task execution thread
- **Before**: `run()` method executes
- **If throws RuntimeException**: Task not executed, `afterExecute` not called

#### afterExecute
- **Called in**: Task execution thread
- **After**: `run()` completes (normal or exception)
- **Not called if**: Task throws `Error`

#### terminated
- **Called**: After all tasks finished and all worker threads shut down
- **Use for**: Cleanup, final logging, statistics finalization

---

## 8.4.1 Example: Thread Pool with Logging and Timing

```java
public class TimingThreadPool extends ThreadPoolExecutor {
    private final ThreadLocal<Long> startTime = new ThreadLocal<>();
    private final Logger log = Logger.getLogger("TimingThreadPool");
    private final AtomicLong numTasks = new AtomicLong();
    private final AtomicLong totalTime = new AtomicLong();
    
    public TimingThreadPool(int corePoolSize, int maximumPoolSize,
                           long keepAliveTime, TimeUnit unit,
                           BlockingQueue<Runnable> workQueue) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, 
              unit, workQueue);
    }
    
    @Override
    protected void beforeExecute(Thread t, Runnable r) {
        super.beforeExecute(t, r);
        log.fine(String.format("Thread %s: start %s", t, r));
        startTime.set(System.nanoTime());
    }
    
    @Override
    protected void afterExecute(Runnable r, Throwable t) {
        try {
            long endTime = System.nanoTime();
            long taskTime = endTime - startTime.get();
            numTasks.incrementAndGet();
            totalTime.addAndGet(taskTime);
            log.fine(String.format("Thread %s: end %s, time=%dns",
                t, r, taskTime));
        } finally {
            super.afterExecute(r, t);
        }
    }
    
    @Override
    protected void terminated() {
        try {
            log.info(String.format("Terminated: avg time=%dns",
                totalTime.get() / numTasks.get()));
        } finally {
            super.terminated();
        }
    }
}
```

**Features**:
- Per-task timing using `ThreadLocal`
- Total task count and time
- Average task time logged on shutdown

**Why ThreadLocal**: Each thread needs own start time

---

## 8.5 Parallelizing Recursive Algorithms

### Loop Parallelization

**Sequential loop**:
```java
void processSequentially(List<Element> elements) {
    for (Element e : elements)
        process(e);
}
```

**Parallel version**:
```java
void processInParallel(Executor exec, List<Element> elements) {
    for (final Element e : elements) {
        exec.execute(new Runnable() {
            public void run() { 
                process(e); 
            }
        });
    }
}
```

**Differences**:
- `processInParallel` returns immediately (tasks queued)
- `processSequentially` waits for all to complete

---

### Waiting for Completion

**Use `invokeAll` to wait**:
```java
List<Callable<Result>> tasks = elements.stream()
    .map(e -> (Callable<Result>) () -> process(e))
    .collect(Collectors.toList());

List<Future<Result>> futures = exec.invokeAll(tasks);
// Blocks until all complete

for (Future<Result> f : futures) {
    Result r = f.get(); // Won't block, already complete
}
```

**Or use `CompletionService`** for results as available

---

### Suitability Criteria

**Good candidates for parallelization**:
- ✅ Iterations are independent
- ✅ Work per iteration is significant (amortizes task overhead)
- ✅ No shared mutable state

**Poor candidates**:
- ❌ Iterations depend on previous results
- ❌ Trivial work per iteration
- ❌ Heavy synchronization needed

---

### Recursive Algorithm Parallelization

**Sequential depth-first traversal**:
```java
public<T> void sequentialRecursive(List<Node<T>> nodes,
                                   Collection<T> results) {
    for (Node<T> n : nodes) {
        results.add(n.compute());
        sequentialRecursive(n.getChildren(), results);
    }
}
```

**Parallel version**:
```java
public<T> void parallelRecursive(final Executor exec,
                                 List<Node<T>> nodes,
                                 final Collection<T> results) {
    for (final Node<T> n : nodes) {
        exec.execute(new Runnable() {
            public void run() {
                results.add(n.compute());
            }
        });
        parallelRecursive(exec, n.getChildren(), results);
    }
}
```

**Characteristics**:
- Traversal still sequential (depth-first order preserved)
- Only `compute()` calls execute in parallel

---

### Waiting for Parallel Recursion Results

```java
public<T> Collection<T> getParallelResults(List<Node<T>> nodes)
        throws InterruptedException {
    ExecutorService exec = Executors.newCachedThreadPool();
    Queue<T> resultQueue = new ConcurrentLinkedQueue<>();
    
    parallelRecursive(exec, nodes, resultQueue);
    
    exec.shutdown();
    exec.awaitTermination(Long.MAX_VALUE, TimeUnit.SECONDS);
    
    return resultQueue;
}
```

**Key points**:
- Use `ConcurrentLinkedQueue` for thread-safe results collection
- Create executor for specific traversal
- Shutdown and await termination
- Results available after `awaitTermination` returns

---

## 8.5.1 Example: Puzzle Solver Framework

### Puzzle Abstraction

```java
public interface Puzzle<P, M> {
    P initialPosition();
    boolean isGoal(P position);
    Set<M> legalMoves(P position);
    P move(P position, M move);
}
```

**Type parameters**:
- `P` = Position type
- `M` = Move type

---

### Node Representation

```java
@Immutable
static class Node<P, M> {
    final P pos;
    final M move;
    final Node<P, M> prev;
    
    Node(P pos, M move, Node<P, M> prev) {
        this.pos = pos;
        this.move = move;
        this.prev = prev;
    }
    
    List<M> asMoveList() {
        List<M> solution = new LinkedList<>();
        for (Node<P, M> n = this; n.move != null; n = n.prev)
            solution.add(0, n.move);
        return solution;
    }
}
```

**Purpose**: Track position and path to reach it

---

### Sequential Solver

```java
public class SequentialPuzzleSolver<P, M> {
    private final Puzzle<P, M> puzzle;
    private final Set<P> seen = new HashSet<>();
    
    public List<M> solve() {
        P pos = puzzle.initialPosition();
        return search(new Node<P, M>(pos, null, null));
    }
    
    private List<M> search(Node<P, M> node) {
        if (!seen.contains(node.pos)) {
            seen.add(node.pos);
            
            if (puzzle.isGoal(node.pos))
                return node.asMoveList();
            
            for (M move : puzzle.legalMoves(node.pos)) {
                P pos = puzzle.move(node.pos, move);
                Node<P, M> child = new Node<>(pos, move, node);
                List<M> result = search(child);
                if (result != null)
                    return result;
            }
        }
        return null;
    }
}
```

**Characteristics**:
- Depth-first search
- Bounded by stack size
- Returns first solution found

---

### Concurrent Solver

```java
public class ConcurrentPuzzleSolver<P, M> {
    private final Puzzle<P, M> puzzle;
    private final ExecutorService exec;
    private final ConcurrentMap<P, Boolean> seen;
    final ValueLatch<Node<P, M>> solution = new ValueLatch<>();
    
    public ConcurrentPuzzleSolver(Puzzle<P, M> puzzle) {
        this.puzzle = puzzle;
        this.exec = Executors.newCachedThreadPool();
        this.seen = new ConcurrentHashMap<>();
    }
    
    public List<M> solve() throws InterruptedException {
        try {
            P p = puzzle.initialPosition();
            exec.execute(newTask(p, null, null));
            
            // Block until solution found
            Node<P, M> solnNode = solution.getValue();
            return (solnNode == null) ? null : solnNode.asMoveList();
        } finally {
            exec.shutdown();
        }
    }
    
    protected Runnable newTask(P p, M m, Node<P,M> n) {
        return new SolverTask(p, m, n);
    }
    
    class SolverTask extends Node<P, M> implements Runnable {
        SolverTask(P pos, M move, Node<P,M> prev) {
            super(pos, move, prev);
        }
        
        public void run() {
            if (solution.isSet() || 
                seen.putIfAbsent(pos, true) != null)
                return; // Already solved or seen
            
            if (puzzle.isGoal(pos))
                solution.setValue(this);
            else
                for (M m : puzzle.legalMoves(pos))
                    exec.execute(newTask(puzzle.move(pos, m), m, this));
        }
    }
}
```

**Key differences from sequential**:
- **Breadth-first** instead of depth-first
- Not bounded by stack size
- Uses thread pool work queue instead of call stack
- Thread-safe with `ConcurrentHashMap` and `ValueLatch`

---

### Result-Bearing Latch

```java
@ThreadSafe
public class ValueLatch<T> {
    @GuardedBy("this") 
    private T value = null;
    private final CountDownLatch done = new CountDownLatch(1);
    
    public boolean isSet() {
        return (done.getCount() == 0);
    }
    
    public synchronized void setValue(T newValue) {
        if (!isSet()) {
            value = newValue;
            done.countDown();
        }
    }
    
    public T getValue() throws InterruptedException {
        done.await();
        synchronized (this) {
            return value;
        }
    }
}
```

**Features**:
- Thread-safe first-wins semantics
- Blocking retrieval
- Non-blocking status check

---

### Handling "No Solution" Case

**Problem**: Original concurrent solver waits forever if no solution exists

**Solution**: Count active tasks

```java
public class PuzzleSolver<P,M> extends ConcurrentPuzzleSolver<P,M> {
    private final AtomicInteger taskCount = new AtomicInteger(0);
    
    protected Runnable newTask(P p, M m, Node<P,M> n) {
        return new CountingSolverTask(p, m, n);
    }
    
    class CountingSolverTask extends SolverTask {
        CountingSolverTask(P pos, M move, Node<P, M> prev) {
            super(pos, move, prev);
            taskCount.incrementAndGet();
        }
        
        public void run() {
            try {
                super.run();
            } finally {
                if (taskCount.decrementAndGet() == 0)
                    solution.setValue(null);  // No solution exists
            }
        }
    }
}
```

**Mechanism**: When last task completes without finding solution, set result to `null`

---

### Additional Termination Conditions

#### 1. **Time Limit**

```java
public List<M> solve(long timeout, TimeUnit unit) 
        throws InterruptedException {
    try {
        P p = puzzle.initialPosition();
        exec.execute(newTask(p, null, null));
        
        Node<P, M> solnNode = solution.getValue(timeout, unit);
        return (solnNode == null) ? null : solnNode.asMoveList();
    } finally {
        exec.shutdown();
    }
}
```

**Requires**: Timed `getValue` in `ValueLatch`

---

#### 2. **Position Limit**

```java
class SolverTask extends Node<P, M> implements Runnable {
    private static final AtomicInteger positionsSearched = 
        new AtomicInteger(0);
    private static final int MAX_POSITIONS = 100_000;
    
    public void run() {
        if (positionsSearched.incrementAndGet() > MAX_POSITIONS) {
            solution.setValue(null); // Give up
            return;
        }
        // ... rest of search logic
    }
}
```

---

#### 3. **Cancellation Mechanism**

```java
public class CancellablePuzzleSolver<P, M> {
    private volatile boolean cancelled = false;
    
    public void cancel() {
        cancelled = true;
        exec.shutdownNow();
    }
    
    class SolverTask extends Node<P, M> implements Runnable {
        public void run() {
            if (cancelled) return;
            // ... rest of logic
        }
    }
}
```

---

## Summary

### Key Takeaways

**Task-Execution Policy Coupling**:
- Not all tasks compatible with all policies
- Document requirements (thread confinement, dependencies, response time)
- Avoid thread starvation deadlock with dependent tasks

**Thread Pool Sizing**:
- Compute-intensive: $N_{cpu} + 1$
- Mixed workload: $N_{cpu} \times U_{cpu} \times (1 + W/C)$
- Consider all resource constraints (memory, connections, handles)
- Don't hard-code sizes

**ThreadPoolExecutor Configuration**:
- **Core/max pool size**: Control thread lifecycle
- **Keep-alive time**: Reclaim idle threads
- **Work queue**: Unbounded, bounded, or synchronous handoff
- **Saturation policy**: Abort, discard, discard-oldest, caller-runs
- **Thread factory**: Naming, logging, exception handling

**Extending ThreadPoolExecutor**:
- Override `beforeExecute`, `afterExecute`, `terminated`
- Add logging, timing, statistics
- Use `ThreadLocal` for per-task state

**Parallelization**:
- Loop iterations: Use `Executor` for independent iterations
- Recursive algorithms: Submit subtasks, use `shutdown`/`awaitTermination`
- Puzzle framework: Breadth-first search with work queue

---

### Common Patterns

| Pattern | Implementation | Use Case |
|---------|---------------|----------|
| Throttling | `CallerRunsPolicy` | Graceful degradation under load |
| Bounded submission | `Semaphore` + task wrapper | Hard limit on in-flight tasks |
| Result collection | `invokeAll` or `CompletionService` | Waiting for parallel tasks |
| Task counting | `AtomicInteger` in wrapper | Detecting completion |
| First-result | `ValueLatch` | Taking first successful result |
| Custom threads | `ThreadFactory` | Naming, logging, monitoring |

---

### Anti-Patterns to Avoid

❌ Hard-coding pool sizes
❌ Using `ThreadLocal` to communicate between tasks
❌ Mixing long/short tasks in same pool
❌ Submitting dependent tasks to bounded pool
❌ `DiscardOldestPolicy` with `PriorityQueue`
❌ Swallowing exceptions in custom thread classes
❌ Not shutting down executors
❌ Unbounded queues without monitoring

---

### Configuration Checklist

When configuring a thread pool, consider:

- [ ] Task characteristics (CPU vs I/O, duration, dependencies)
- [ ] Pool size (based on CPU count, utilization target, W/C ratio)
- [ ] Queue type (bounded, unbounded, synchronous)
- [ ] Queue capacity (if bounded)
- [ ] Saturation policy (abort, discard, caller-runs)
- [ ] Thread factory (naming, exception handling)
- [ ] Keep-alive time
- [ ] Resource constraints (memory, connections, handles)
- [ ] Monitoring (logging, metrics, alerts)

---

# Chapter 9 – GUI Applications

GUI applications have unique threading challenges. To maintain safety, certain tasks must run in the GUI event thread, but long-running tasks cannot execute there without making the UI unresponsive. Understanding how to properly divide work between the event thread and background threads is essential for building responsive, thread-safe GUI applications.

---

## Why GUIs Are Single-threaded

Nearly all GUI toolkits—including Swing, SWT, Qt, MacOS Cocoa, and X Windows—use a **single-threaded event queue model**. A dedicated **event dispatch thread (EDT)** fetches events from a queue and dispatches them to application-defined handlers.

### Failed Attempts at Multithreaded GUIs

Multithreaded GUI frameworks have been attempted but consistently failed due to:

1. **Inconsistent Lock Ordering**: User actions "bubble up" from OS to application, while application actions "bubble down" to the OS. This bidirectional flow accessing the same GUI objects leads to deadlock.

2. **MVC Pattern Complications**: The Model-View-Controller pattern exacerbates lock ordering issues:
   - Controller → Model → View (notifications)
   - Controller → View → Model (queries)

   This creates circular dependencies prone to deadlock.

3. **Complexity Barrier**: As Graham Hamilton noted, multithreaded GUI toolkits require intimate knowledge of the entire toolkit structure, which doesn't scale to widespread use.

---

## Thread Confinement in Swing

Single-threaded GUI frameworks achieve thread safety through **thread confinement**—all GUI objects are accessed exclusively from the event thread.

### The Swing Single-thread Rule

> Swing components and models should be created, modified, and queried only from the event-dispatching thread.

### Exceptions to the Rule

A few Swing methods are thread-safe and callable from any thread:

- `SwingUtilities.isEventDispatchThread()` – check if current thread is EDT
- `SwingUtilities.invokeLater(Runnable)` – schedule task on EDT
- `SwingUtilities.invokeAndWait(Runnable)` – schedule and block until complete
- Repaint/revalidation requests
- Listener add/remove methods

---

## Sequential Event Processing

Because GUI tasks run on a single thread, they execute sequentially—one task finishes before the next begins. This simplifies task code (no interference worries) but creates a critical constraint:

**Tasks in the event thread must return control quickly.**

If a lengthy task runs in the event thread:
- The UI appears frozen
- Users cannot click buttons (even "Cancel")
- No visual feedback occurs

---

## Implementing SwingUtilities with Executor

The threading methods in `SwingUtilities` are conceptually similar to an `Executor`:

```java
public class SwingUtilities {
    private static final ExecutorService exec =
        Executors.newSingleThreadExecutor(new SwingThreadFactory());
    private static volatile Thread swingThread;

    private static class SwingThreadFactory implements ThreadFactory {
        public Thread newThread(Runnable r) {
            swingThread = new Thread(r);
            return swingThread;
        }
    }

    public static boolean isEventDispatchThread() {
        return Thread.currentThread() == swingThread;
    }

    public static void invokeLater(Runnable task) {
        exec.execute(task);
    }

    public static void invokeAndWait(Runnable task)
            throws InterruptedException, InvocationTargetException {
        Future f = exec.submit(task);
        try {
            f.get();
        } catch (ExecutionException e) {
            throw new InvocationTargetException(e);
        }
    }
}
```

---

## GuiExecutor Pattern

A convenient `Executor` that delegates to `SwingUtilities`:

```java
public class GuiExecutor extends AbstractExecutorService {
    private static final GuiExecutor instance = new GuiExecutor();

    private GuiExecutor() { }

    public static GuiExecutor instance() { return instance; }

    public void execute(Runnable r) {
        if (SwingUtilities.isEventDispatchThread())
            r.run();
        else
            SwingUtilities.invokeLater(r);
    }
}
```

---

## Short-running GUI Tasks

For simple, short-running tasks, the entire action stays in the event thread:

```java
final Random random = new Random();
final JButton button = new JButton("Change Color");

button.addActionListener(new ActionListener() {
    public void actionPerformed(ActionEvent e) {
        button.setBackground(new Color(random.nextInt()));
    }
});
```

**Control flow**: Event originates in toolkit → delivered to listener → modifies GUI → control never leaves EDT.

### Model-View Pattern

With data models (like `TableModel`), the flow is:
1. Action listener updates the model
2. Model fires change events (`fireXxx` methods)
3. View receives notification and queries model
4. View updates display

All of this happens within the event thread.

---

## Long-running GUI Tasks

Tasks that take too long for the event thread must run in background threads. Use a cached thread pool:

```java
ExecutorService backgroundExec = Executors.newCachedThreadPool();

button.addActionListener(new ActionListener() {
    public void actionPerformed(ActionEvent e) {
        backgroundExec.execute(new Runnable() {
            public void run() { doBigComputation(); }
        });
    }
});
```

### Thread Hopping with User Feedback

Long-running tasks typically involve "thread hopping" between EDT and background threads:

```java
button.addActionListener(new ActionListener() {
    public void actionPerformed(ActionEvent e) {
        button.setEnabled(false);           // 1. Update UI (EDT)
        label.setText("busy");

        backgroundExec.execute(new Runnable() {
            public void run() {
                try {
                    doBigComputation();      // 2. Long task (background)
                } finally {
                    GuiExecutor.instance().execute(new Runnable() {
                        public void run() {
                            button.setEnabled(true);   // 3. Update UI (EDT)
                            label.setText("idle");
                        }
                    });
                }
            }
        });
    }
});
```

---

## Cancellation

Use `Future` for managing cancellable tasks:

```java
Future<?> runningTask = null;  // thread-confined to EDT

startButton.addActionListener(new ActionListener() {
    public void actionPerformed(ActionEvent e) {
        if (runningTask == null) {
            runningTask = backgroundExec.submit(new Runnable() {
                public void run() {
                    while (moreWork()) {
                        if (Thread.currentThread().isInterrupted()) {
                            cleanUpPartialWork();
                            break;
                        }
                        doSomeWork();
                    }
                }
            });
        }
    }
});

cancelButton.addActionListener(new ActionListener() {
    public void actionPerformed(ActionEvent event) {
        if (runningTask != null)
            runningTask.cancel(true);
    }
});
```

---

## BackgroundTask Framework

A reusable framework combining `FutureTask` with EDT callbacks:

```java
abstract class BackgroundTask<V> implements Runnable, Future<V> {
    private final FutureTask<V> computation = new Computation();

    private class Computation extends FutureTask<V> {
        public Computation() {
            super(new Callable<V>() {
                public V call() throws Exception {
                    return BackgroundTask.this.compute();
                }
            });
        }

        protected final void done() {
            GuiExecutor.instance().execute(new Runnable() {
                public void run() {
                    V value = null;
                    Throwable thrown = null;
                    boolean cancelled = false;
                    try {
                        value = get();
                    } catch (ExecutionException e) {
                        thrown = e.getCause();
                    } catch (CancellationException e) {
                        cancelled = true;
                    } catch (InterruptedException consumed) {
                    } finally {
                        onCompletion(value, thrown, cancelled);
                    }
                }
            });
        }
    }

    protected void setProgress(final int current, final int max) {
        GuiExecutor.instance().execute(new Runnable() {
            public void run() { onProgress(current, max); }
        });
    }

    // Called in background thread
    protected abstract V compute() throws Exception;

    // Called in event thread
    protected void onCompletion(V result, Throwable exception,
                                boolean cancelled) { }
    protected void onProgress(int current, int max) { }
}
```

### Features

- **Cancellation**: Check `isCancelled()` in `compute()`
- **Progress**: Call `setProgress()` from background thread
- **Completion**: Override `onCompletion()` for EDT notification

---

## SwingWorker (Java 6+)

Java 6 introduced `SwingWorker`, which provides similar functionality to `BackgroundTask`:
- Cancellation support
- Completion notification
- Progress indication

---

## Shared Data Models

### Thread-safe Data Models

When multiple threads need access to data:
- Use thread-safe collections (e.g., `ConcurrentHashMap`)
- Trade-off: May not provide consistent snapshots
- Versioned structures like `CopyOnWriteArrayList` work when traversals >> modifications

### Split Data Models

For complex applications, use a **split-model design**:

| Model | Thread | Characteristics |
|-------|--------|-----------------|
| **Presentation Model** | Event thread only | Swing's `TableModel`, `TreeModel` |
| **Shared Model** | Thread-safe | Application domain data |

**Synchronization approaches**:
1. **Snapshot**: Embed relevant state in update messages
2. **Incremental**: Send only changed data (more efficient for large models)

---

## Other Single-threaded Subsystems

Thread confinement applies beyond GUIs:

- **Native libraries**: Some require all access from the same thread
- **Pattern**: Create a dedicated thread/executor, use a proxy to submit tasks

```java
// Proxy pattern for thread-confined resource
public class NativeLibraryProxy {
    private final ExecutorService executor =
        Executors.newSingleThreadExecutor();

    public Result doOperation(Params params) {
        Future<Result> future = executor.submit(() ->
            nativeLibrary.operation(params));
        return future.get();  // Block for result
    }
}
```

---

## Summary

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Single-threaded GUI** | All GUI toolkits use single event thread due to deadlock issues in multithreaded attempts |
| **Thread confinement** | GUI objects accessed only from EDT |
| **Sequential processing** | Tasks run one at a time; long tasks freeze UI |
| **Thread hopping** | Alternate between EDT and background threads |

### Threading Rules for GUI Applications

**In the Event Thread**:
- All GUI component access
- Short-running tasks only
- Starting/scheduling background tasks
- Updating UI after background completion

**In Background Threads**:
- Long-running computations
- I/O operations
- Network requests
- File system operations

### Patterns

| Pattern | Purpose |
|---------|---------|
| `invokeLater` | Schedule task on EDT (non-blocking) |
| `invokeAndWait` | Schedule task on EDT (blocking) |
| `GuiExecutor` | Executor that delegates to EDT |
| `BackgroundTask` | Reusable framework for cancellable background tasks |
| `SwingWorker` | Built-in Java 6+ solution |
| **Split model** | Separate presentation model (EDT) from shared model (thread-safe) |

### Anti-Patterns to Avoid

❌ Accessing Swing components from background threads
❌ Running long operations in the event thread
❌ Blocking the EDT with `invokeAndWait` from EDT itself
❌ Ignoring thread confinement for data models
❌ Not providing cancellation for long-running tasks
❌ Forgetting to update UI after background task completes

### Design Guidelines

1. **Keep the EDT responsive** – never block it with long operations
2. **Confine GUI objects** – access only from EDT
3. **Use executors** – `newCachedThreadPool()` for background tasks
4. **Support cancellation** – use `Future` and interruption
5. **Provide feedback** – progress indicators and completion notifications
6. **Consider split models** – when sharing data between EDT and application threads

---

# Chapter 10 – Avoiding Liveness Hazards

There is often a tension between safety and liveness. We use locking to ensure thread safety, but indiscriminate use of locking can cause lock-ordering deadlocks. Java applications do not recover from deadlock, so it is essential to design systems that preclude the conditions that cause it.

---

## Deadlock

Deadlock is illustrated by the classic **dining philosophers** problem: five philosophers share five chopsticks (one between each pair). Each needs two chopsticks to eat. If each grabs the left chopstick and waits for the right one, all starve—this is deadlock.

### Definition

When thread A holds lock L and tries to acquire lock M, while thread B holds M and tries to acquire L, both threads wait forever. This is the simplest case of deadlock (or **deadly embrace**).

**Graph model**: Think of threads as nodes in a directed graph where edges represent "waiting for a resource held by". If the graph is cyclic, there is a deadlock.

### JVM vs Database Handling

| System | Deadlock Handling |
|--------|-------------------|
| **Database** | Detects cycles, picks a victim, aborts transaction, allows retry |
| **JVM** | No recovery—deadlocked threads are permanently out of commission |

Deadlocks rarely manifest immediately. They often appear under heavy production load at the worst possible time.

---

## Lock-ordering Deadlocks

```java
// Warning: deadlock-prone!
public class LeftRightDeadlock {
    private final Object left = new Object();
    private final Object right = new Object();

    public void leftRight() {
        synchronized (left) {
            synchronized (right) {
                doSomething();
            }
        }
    }

    public void rightLeft() {
        synchronized (right) {
            synchronized (left) {
                doSomethingElse();
            }
        }
    }
}
```

**Problem**: Two threads acquire the same locks in different orders.

**Solution**: If all threads acquire locks L and M in the same order, there will be no cyclic dependency and no deadlock.

> A program will be free of lock-ordering deadlocks if all threads acquire the locks they need in a fixed global order.

---

## Dynamic Lock Order Deadlocks

Sometimes lock order depends on runtime arguments:

```java
// Warning: deadlock-prone!
public void transferMoney(Account fromAccount,
                          Account toAccount,
                          DollarAmount amount)
        throws InsufficientFundsException {
    synchronized (fromAccount) {
        synchronized (toAccount) {
            if (fromAccount.getBalance().compareTo(amount) < 0)
                throw new InsufficientFundsException();
            else {
                fromAccount.debit(amount);
                toAccount.credit(amount);
            }
        }
    }
}
```

**Deadlock scenario**:
```java
A: transferMoney(myAccount, yourAccount, 10);
B: transferMoney(yourAccount, myAccount, 20);
```

Thread A locks `myAccount`, waits for `yourAccount`. Thread B locks `yourAccount`, waits for `myAccount`.

### Solution: Induce Lock Ordering

Use `System.identityHashCode` to create a consistent ordering:

```java
private static final Object tieLock = new Object();

public void transferMoney(final Account fromAcct,
                          final Account toAcct,
                          final DollarAmount amount)
        throws InsufficientFundsException {
    class Helper {
        public void transfer() throws InsufficientFundsException {
            if (fromAcct.getBalance().compareTo(amount) < 0)
                throw new InsufficientFundsException();
            else {
                fromAcct.debit(amount);
                toAcct.credit(amount);
            }
        }
    }

    int fromHash = System.identityHashCode(fromAcct);
    int toHash = System.identityHashCode(toAcct);

    if (fromHash < toHash) {
        synchronized (fromAcct) {
            synchronized (toAcct) {
                new Helper().transfer();
            }
        }
    } else if (fromHash > toHash) {
        synchronized (toAcct) {
            synchronized (fromAcct) {
                new Helper().transfer();
            }
        }
    } else {
        synchronized (tieLock) {  // Tie-breaker for hash collisions
            synchronized (fromAcct) {
                synchronized (toAcct) {
                    new Helper().transfer();
                }
            }
        }
    }
}
```

**Key points**:
- Order locks by hash code (or any unique comparable key like account number)
- Use a tie-breaking lock for rare hash collisions
- Hash collisions are vanishingly rare, so the tie-breaker rarely becomes a bottleneck

---

## Deadlocks Between Cooperating Objects

Deadlocks can occur across object boundaries:

```java
// Warning: deadlock-prone!
class Taxi {
    @GuardedBy("this") private Point location, destination;
    private final Dispatcher dispatcher;

    public synchronized void setLocation(Point location) {
        this.location = location;
        if (location.equals(destination))
            dispatcher.notifyAvailable(this);  // Calls alien method while holding lock
    }
}

class Dispatcher {
    @GuardedBy("this") private final Set<Taxi> taxis;
    @GuardedBy("this") private final Set<Taxi> availableTaxis;

    public synchronized void notifyAvailable(Taxi taxi) {
        availableTaxis.add(taxi);
    }

    public synchronized Image getImage() {
        Image image = new Image();
        for (Taxi t : taxis)
            image.drawMarker(t.getLocation());  // Calls into Taxi while holding lock
        return image;
    }
}
```

**Problem**:
- `setLocation`: acquires Taxi lock → Dispatcher lock
- `getImage`: acquires Dispatcher lock → Taxi lock

**Warning sign**: Calling an **alien method** (a method you don't control) while holding a lock.

> Invoking an alien method with a lock held is asking for liveness trouble.

---

## Open Calls

An **open call** is a method call made with no locks held.

### Benefits of Open Calls

- Easier to analyze for deadlock freedom
- More composable classes
- Analogous to encapsulation for thread safety

### Refactored Taxi and Dispatcher

```java
@ThreadSafe
class Taxi {
    @GuardedBy("this") private Point location, destination;
    private final Dispatcher dispatcher;

    public synchronized Point getLocation() {
        return location;
    }

    public void setLocation(Point location) {
        boolean reachedDestination;
        synchronized (this) {
            this.location = location;
            reachedDestination = location.equals(destination);
        }
        if (reachedDestination)
            dispatcher.notifyAvailable(this);  // Open call
    }
}

@ThreadSafe
class Dispatcher {
    @GuardedBy("this") private final Set<Taxi> taxis;
    @GuardedBy("this") private final Set<Taxi> availableTaxis;

    public synchronized void notifyAvailable(Taxi taxi) {
        availableTaxis.add(taxi);
    }

    public Image getImage() {
        Set<Taxi> copy;
        synchronized (this) {
            copy = new HashSet<Taxi>(taxis);  // Copy under lock
        }
        Image image = new Image();
        for (Taxi t : copy)
            image.drawMarker(t.getLocation());  // Open call
        return image;
    }
}
```

**Trade-off**: Loss of atomicity. The refactored `getImage` fetches locations at slightly different times rather than as a complete snapshot. Often this is acceptable.

> Strive to use open calls throughout your program.

---

## Resource Deadlocks

Threads can deadlock waiting for resources, not just locks.

### Connection Pool Deadlock

If tasks require connections to two databases (D1 and D2) and don't request them in the same order:
- Thread A holds D1 connection, waits for D2
- Thread B holds D2 connection, waits for D1

### Thread-starvation Deadlock

A task that submits another task and waits for its result in a single-threaded `Executor` will wait forever:

```java
ExecutorService exec = Executors.newSingleThreadExecutor();

exec.submit(() -> {
    Future<?> inner = exec.submit(() -> doWork());
    inner.get();  // Deadlock! Outer task blocks the only thread
});
```

> Tasks that wait for results of other tasks are the primary source of thread-starvation deadlock. Bounded pools and interdependent tasks do not mix well.

---

## Avoiding and Diagnosing Deadlocks

### Prevention Strategies

1. **Minimize lock acquisition**: A program that never acquires more than one lock at a time cannot experience lock-ordering deadlock

2. **Document lock ordering**: Make lock ordering part of your design

3. **Use open calls**: Makes it easier to identify code paths that acquire multiple locks

4. **Two-part audit strategy**:
   - Identify where multiple locks could be acquired
   - Perform global analysis to ensure consistent ordering

### Timed Lock Attempts

Use explicit `Lock` with `tryLock` timeout instead of intrinsic locks:

```java
if (lock.tryLock(timeout, TimeUnit.MILLISECONDS)) {
    try {
        // do work
    } finally {
        lock.unlock();
    }
} else {
    // Handle timeout: log, back off, retry
}
```

**Benefits**:
- Regain control when something goes wrong
- Can release locks, back off, and retry
- Log useful diagnostic information

---

## Thread Dumps for Deadlock Analysis

The JVM can identify deadlocks via thread dumps.

### Triggering a Thread Dump

| Platform | Method |
|----------|--------|
| Unix | `kill -3 <pid>` or `Ctrl-\` |
| Windows | `Ctrl-Break` |
| IDEs | Built-in thread dump feature |

### Example Deadlock in Thread Dump

```
Found one Java-level deadlock:
=============================
"ApplicationServerThread":
  waiting to lock monitor 0x080f0cdc (a MumbleDBConnection),
  which is held by "ApplicationServerThread"
"ApplicationServerThread":
  waiting to lock monitor 0x080f0ed4 (a MumbleDBCallableStatement),
  which is held by "ApplicationServerThread"

Java stack information for the threads listed above:
"ApplicationServerThread":
  at MumbleDBConnection.remove_statement
  - waiting to lock <0x650f7f30> (a MumbleDBConnection)
  at MumbleDBStatement.close
  - locked <0x6024ffb0> (a MumbleDBCallableStatement)
  ...
"ApplicationServerThread":
  at MumbleDBCallableStatement.sendBatch
  - waiting to lock <0x6024ffb0> (a MumbleDBCallableStatement)
  at MumbleDBConnection.commit
  - locked <0x650f7f30> (a MumbleDBConnection)
```

**Note**: Java 5 thread dumps don't show explicit `Lock` information. Java 6+ includes support for explicit locks but with less precise acquisition location.

---

## Other Liveness Hazards

### Starvation

A thread is perpetually denied resources it needs to make progress.

**Causes**:
- Inappropriate use of thread priorities
- Holding a lock while executing non-terminating constructs (infinite loops, resource waits)

**Advice**:
> Avoid the temptation to use thread priorities, since they increase platform dependence and can cause liveness problems. Most concurrent applications can use the default priority for all threads.

Thread priorities are merely scheduling hints and map differently across platforms.

### Poor Responsiveness

**Causes**:
- CPU-intensive background tasks competing with event thread
- Holding locks for long periods (e.g., iterating large collections)

**Solution**: Lower priority of truly background tasks; minimize lock hold times.

### Livelock

A thread is not blocked but cannot make progress because it keeps retrying an operation that always fails.

**Example: Poison Message Problem**

```
1. Message handler fails processing message
2. Transaction rolls back, message goes back to queue head
3. Message is dequeued again
4. Handler fails again
5. Repeat forever...
```

The thread is not blocked, but it never makes progress.

**Example: Overly Polite Threads**

Two threads keep changing state in response to each other, like two people in a hallway who keep stepping aside into each other's path.

**Solution**: Introduce randomness and exponential back-off (like Ethernet collision handling):

```java
int attempts = 0;
while (!success && attempts < MAX_ATTEMPTS) {
    try {
        success = tryOperation();
    } catch (ConflictException e) {
        attempts++;
        // Random back-off with exponential increase
        Thread.sleep(random.nextInt(baseDelay * (1 << attempts)));
    }
}
```

---

## Summary

### Liveness Hazard Types

| Hazard | Description | Solution |
|--------|-------------|----------|
| **Deadlock** | Threads wait forever for locks held by each other | Consistent lock ordering, open calls |
| **Resource Deadlock** | Threads wait forever for resources | Consistent resource acquisition order |
| **Thread-starvation Deadlock** | Task waits for result of task in same bounded pool | Don't mix interdependent tasks with bounded pools |
| **Starvation** | Thread perpetually denied resources | Avoid thread priority manipulation |
| **Livelock** | Thread not blocked but cannot progress | Random back-off and retry |

### Deadlock Prevention Checklist

- [ ] Acquire locks in a fixed global order
- [ ] Use open calls (no alien method calls while holding locks)
- [ ] Minimize the number of locks held simultaneously
- [ ] Shrink synchronized blocks to minimum necessary
- [ ] Use timed `tryLock` for recovery capability
- [ ] Document lock ordering protocols
- [ ] Audit code for multiple lock acquisitions

### Key Patterns

| Pattern | Purpose |
|---------|---------|
| **Lock ordering by hash code** | Consistent ordering for dynamic lock acquisition |
| **Tie-breaking lock** | Handle hash collisions in lock ordering |
| **Open calls** | Prevent alien method deadlocks |
| **Copy-then-iterate** | Release lock before iterating |
| **Timed tryLock** | Detect and recover from potential deadlocks |
| **Random back-off** | Prevent livelock |

### Anti-Patterns to Avoid

❌ Acquiring locks in inconsistent order
❌ Calling alien methods while holding locks
❌ Using synchronized methods when smaller blocks suffice
❌ Submitting interdependent tasks to bounded thread pools
❌ Manipulating thread priorities
❌ Holding locks during long operations
❌ Retrying failed operations without back-off

### Design Principles

1. **Lock ordering must be part of your design** – not an afterthought
2. **Open calls are analogous to encapsulation** – they make analysis tractable
3. **The left hand needs to know what the right hand is doing** – global analysis required
4. **Deadlocks manifest under load** – testing may not reveal them
5. **There is no recovery from deadlock** – prevention is the only option

---

# Chapter 11 – Performance and Scalability

One of the primary reasons to use threads is to improve performance—utilizing processing resources more effectively and improving responsiveness. However, many techniques for improving performance also increase complexity, raising the likelihood of safety and liveness failures.

> First make your program right, then make it fast—and then only if your performance requirements and measurements tell you it needs to be faster.

---

## Thinking About Performance

**Improving performance** means doing more work with fewer resources. Performance can be limited by various resources:
- CPU cycles (CPU-bound)
- Memory
- Network bandwidth
- I/O bandwidth
- Database requests

### Threading Costs vs Benefits

Threading always introduces costs:
- Coordination overhead (locking, signaling, memory synchronization)
- Context switching
- Thread creation and teardown
- Scheduling overhead

When threading is employed effectively, these costs are outweighed by greater throughput, responsiveness, or capacity.

---

## Performance vs Scalability

| Aspect | Measures | Focus |
|--------|----------|-------|
| **Performance** | Service time, latency | "How fast" – processing speed of individual units |
| **Scalability** | Throughput, capacity | "How much" – work done with given/additional resources |

**Scalability**: The ability to improve throughput or capacity when additional computing resources are added.

### Key Insight: Performance and Scalability Can Conflict

- **Performance tuning**: Do the same work with less effort (caching, better algorithms)
- **Scalability tuning**: Parallelize to use more resources effectively

Many single-threaded performance tricks are bad for scalability.

### The Three-Tier Example

A monolithic application outperforms a distributed three-tier system for the first unit of work (no network latency, no coordination overhead). But when the monolithic system reaches capacity, scaling is prohibitively difficult.

> We often accept performance costs of longer service time so that our application can scale to handle greater load.

For server applications, scalability is usually more important than raw speed.

---

## Evaluating Performance Tradeoffs

Before optimizing, ask:
- What do you mean by "faster"?
- Under what conditions will this be faster? (light/heavy load, small/large data)
- How often will these conditions arise?
- What hidden costs are you trading? (development risk, maintenance complexity)

> Avoid premature optimization. First make it right, then make it fast—if it is not already fast enough.

### The Concurrency Bug Risk

> The quest for performance is probably the single greatest source of concurrency bugs.

The belief that synchronization is "too slow" has led to dangerous idioms like double-checked locking.

> Measure, don't guess.

---

## Amdahl's Law

Amdahl's law describes how much a program can theoretically be sped up by additional computing resources:

$$Speedup \leq \frac{1}{F + \frac{(1-F)}{N}}$$

Where:
- **F** = fraction that must be executed serially
- **N** = number of processors

As N → ∞, maximum speedup converges to **1/F**.

### Implications

| Serial Fraction | Max Speedup | Notes |
|-----------------|-------------|-------|
| 50% | 2x | No matter how many processors |
| 10% | 10x | Theoretical maximum |
| 1% | 100x | Rare to achieve in practice |

### Utilization Under Amdahl's Law

With 10% serialization:
- 10 processors: speedup of 5.3x (53% utilization)
- 100 processors: speedup of 9.2x (9% utilization)

Even small serialization percentages devastate scalability at high processor counts.

### Sources of Serialization

```java
public class WorkerThread extends Thread {
    private final BlockingQueue<Runnable> queue;

    public void run() {
        while (true) {
            try {
                Runnable task = queue.take();  // Serialization point!
                task.run();
            } catch (InterruptedException e) {
                break;
            }
        }
    }
}
```

**Common serialization sources**:
- Shared work queues
- Result handling (log files, shared data structures)
- Any shared data structure access

> All concurrent applications have some sources of serialization; if you think yours does not, look again.

### Framework Comparison Example

| Implementation | Scalability | Why |
|----------------|-------------|-----|
| `ConcurrentLinkedQueue` | Excellent | Non-blocking algorithm, only pointer updates serialized |
| `synchronizedList(LinkedList)` | Poor | Entire operation serialized, heavy contention |

The synchronized version shows improvement up to ~3 threads, then degrades as contention dominates.

### Thinking "In the Limit"

When evaluating algorithms, consider what happens with hundreds or thousands of processors:
- **Lock splitting** (one lock → two): Limited improvement
- **Lock striping** (one lock → many): Promising scalability

---

## Costs Introduced by Threads

### Context Switching

When there are more runnable threads than CPUs, the OS preempts threads, causing context switches.

**Costs**:
- Saving/restoring execution context
- OS and JVM data structure manipulation
- Cache misses (new thread's data not in cache)

**Rule of thumb**: A context switch costs **5,000–10,000 clock cycles** (several microseconds).

**Monitoring**:
- Unix: `vmstat` (context switches, kernel time)
- Windows: `perfmon`

High kernel usage (>10%) often indicates heavy scheduling activity from blocking or lock contention.

### Memory Synchronization

Synchronization costs come from:
- **Memory barriers**: Flush/invalidate caches, stall pipelines
- **Compiler optimization inhibition**: Operations can't be reordered across barriers

**Uncontended vs Contended**:

| Type | Cost | Notes |
|------|------|-------|
| Uncontended | 20–250 clock cycles | Optimized by JVM |
| Contended | Much higher | May involve OS, context switches |

### JVM Optimizations

**Lock elision**: JVM removes locks on thread-local objects:

```java
public String getStoogeNames() {
    List<String> stooges = new Vector<String>();  // Thread-local
    stooges.add("Moe");
    stooges.add("Larry");
    stooges.add("Curly");
    return stooges.toString();
}
// JVM can eliminate all 4 lock acquisitions
```

**Lock coarsening**: JVM merges adjacent synchronized blocks on the same lock.

> Don't worry excessively about the cost of uncontended synchronization. Focus optimization efforts on areas where lock contention actually occurs.

### Blocking

When locking is contended, JVM can:
1. **Spin-wait**: Repeatedly try to acquire (good for short waits)
2. **Suspend**: OS context switch (good for long waits)

Suspension causes **two additional context switches** plus cache effects.

---

## Reducing Lock Contention

> The principal threat to scalability in concurrent applications is the exclusive resource lock.

**Two factors determine contention likelihood**:
1. How often the lock is requested
2. How long it is held once acquired

**Three ways to reduce lock contention**:
1. Reduce duration locks are held
2. Reduce frequency locks are requested
3. Replace exclusive locks with more concurrent alternatives

---

### 1. Narrowing Lock Scope ("Get in, Get out")

Hold locks as briefly as possible.

**Before** – holding lock too long:

```java
@ThreadSafe
public class AttributeStore {
    @GuardedBy("this")
    private final Map<String, String> attributes = new HashMap<>();

    public synchronized boolean userLocationMatches(String name, String regexp) {
        String key = "users." + name + ".location";  // No lock needed
        String location = attributes.get(key);        // Needs lock
        if (location == null)
            return false;
        else
            return Pattern.matches(regexp, location); // No lock needed
    }
}
```

**After** – minimized lock scope:

```java
@ThreadSafe
public class BetterAttributeStore {
    @GuardedBy("this")
    private final Map<String, String> attributes = new HashMap<>();

    public boolean userLocationMatches(String name, String regexp) {
        String key = "users." + name + ".location";
        String location;
        synchronized (this) {
            location = attributes.get(key);  // Only this needs the lock
        }
        if (location == null)
            return false;
        else
            return Pattern.matches(regexp, location);
    }
}
```

**Even better**: Delegate to a thread-safe `ConcurrentHashMap`, eliminating explicit synchronization entirely.

> A synchronized block can be too small—operations that must be atomic need to stay together.

---

### 2. Reducing Lock Granularity (Lock Splitting)

Use separate locks to guard independent state variables.

**Before** – single lock guards independent state:

```java
@ThreadSafe
public class ServerStatus {
    @GuardedBy("this") public final Set<String> users;
    @GuardedBy("this") public final Set<String> queries;

    public synchronized void addUser(String u) { users.add(u); }
    public synchronized void addQuery(String q) { queries.add(q); }
    public synchronized void removeUser(String u) { users.remove(u); }
    public synchronized void removeQuery(String q) { queries.remove(q); }
}
```

**After** – split locks for independent state:

```java
@ThreadSafe
public class ServerStatus {
    @GuardedBy("users") public final Set<String> users;
    @GuardedBy("queries") public final Set<String> queries;

    public void addUser(String u) {
        synchronized (users) { users.add(u); }
    }
    public void addQuery(String q) {
        synchronized (queries) { queries.add(q); }
    }
}
```

**When it helps**: Moderate contention → mostly uncontended locks (best outcome).

---

### 3. Lock Striping

Partition locking on a variable-sized set of independent objects.

**Example**: `ConcurrentHashMap` uses 16 locks, each guarding 1/16 of hash buckets:

```java
@ThreadSafe
public class StripedMap {
    private static final int N_LOCKS = 16;
    private final Node[] buckets;
    private final Object[] locks;

    public StripedMap(int numBuckets) {
        buckets = new Node[numBuckets];
        locks = new Object[N_LOCKS];
        for (int i = 0; i < N_LOCKS; i++)
            locks[i] = new Object();
    }

    private final int hash(Object key) {
        return Math.abs(key.hashCode() % buckets.length);
    }

    public Object get(Object key) {
        int hash = hash(key);
        synchronized (locks[hash % N_LOCKS]) {
            for (Node m = buckets[hash]; m != null; m = m.next)
                if (m.key.equals(key))
                    return m.value;
        }
        return null;
    }

    public void clear() {
        for (int i = 0; i < buckets.length; i++) {
            synchronized (locks[i % N_LOCKS]) {
                buckets[i] = null;
            }
        }
    }
}
```

**Trade-off**: Locking the entire collection (e.g., for rehashing) requires acquiring all stripe locks.

---

### 4. Avoiding Hot Fields

**Hot field**: A variable that every operation must access, limiting lock granularity.

**Example**: Caching `size()` in a counter

```java
// Single-threaded optimization becomes scalability liability
private int count;  // Updated on every put/remove

public int size() {
    return count;  // O(1) but creates hot field
}
```

**ConcurrentHashMap solution**: Maintain separate count per stripe, enumerate on `size()` call.

---

### 5. Alternatives to Exclusive Locks

| Alternative | Use Case |
|-------------|----------|
| `ReadWriteLock` | Read-mostly data (multiple readers, single writer) |
| Immutable objects | Read-only data (no locking needed) |
| Atomic variables | Hot fields like counters, sequence generators |
| Concurrent collections | Replace synchronized wrappers |

> Atomic variables reduce the cost of updating hot fields, but don't eliminate it. Changing your algorithm to have fewer hot fields might improve scalability even more.

---

## Monitoring CPU Utilization

**Tools**:
- Unix: `vmstat`, `mpstat`, `iostat`
- Windows: `perfmon`

### Diagnosing Underutilized CPUs

| Symptom | Likely Cause | Solution |
|---------|--------------|----------|
| Asymmetric CPU usage | Work concentrated in few threads | Find more parallelism |
| Low utilization, low load | Insufficient load | Increase test load |
| Low utilization, high I/O | I/O-bound | Optimize I/O, add async processing |
| Low utilization, external waits | External service bottleneck | Profile external dependencies |
| Low utilization, lock contention | Contended locks | Reduce lock scope/granularity |

**Thread dump sampling**: Trigger a few thread dumps; heavily contended locks frequently appear as "waiting to lock monitor..."

---

## Object Pooling: Just Say No

Object pooling was a workaround for slow allocation in early JVMs.

**Modern reality**:
- `new Object()` in HotSpot ≈ **10 machine instructions**
- Java allocation is now **faster than C's `malloc`**

**Why pooling hurts concurrency**:

| Approach | Coordination Required |
|----------|----------------------|
| Allocation | Minimal (thread-local allocation blocks) |
| Object pool | Synchronization on pool access |

> Blocking a thread due to lock contention is hundreds of times more expensive than an allocation.

> Allocating objects is usually cheaper than synchronizing.

---

## Comparing Map Performance

| Implementation | Scalability | Reason |
|----------------|-------------|--------|
| `ConcurrentHashMap` | Excellent | Lock striping, no locking for most reads |
| `ConcurrentSkipListMap` | Excellent | Lock-free algorithms |
| `synchronizedMap(HashMap)` | Poor | Single lock for entire map |
| `synchronizedMap(TreeMap)` | Poor | Single lock for entire map |

**Observed behavior**:
- Concurrent collections: Throughput improves with threads up to CPU count, then plateaus
- Synchronized collections: Performance comparable at 1 thread, degrades severely at 2+ threads

---

## Reducing Context Switch Overhead

**Example**: Logging approaches

| Approach | Characteristics |
|----------|-----------------|
| Inline logging | Each thread writes directly; I/O blocks, lock contention on stream |
| Background logging | Threads queue messages; dedicated thread handles I/O |

### Why Background Logging Wins

**Inline logging problems**:
- Thread blocks on I/O → context switch
- Lock contention on output stream → more blocking
- Longer lock hold times → more contention

**Background logging benefits**:
- Request threads never block on I/O
- Queue put is lightweight (less likely to block)
- Single writer eliminates output stream contention
- Straight-line code path instead of complex blocking path

**Analogy**: Bucket brigade vs. individuals running with buckets
- Bucket brigade: Constant flow, each worker does one job continuously
- Individuals: Contention at source and destination, constant mode switching

> Just as interruptions are disruptive to humans, blocking and context switching are disruptive to threads.

---

## Summary

### Key Metrics

| Metric | Description | Server Priority |
|--------|-------------|-----------------|
| Throughput | Work completed per unit time | High |
| Scalability | Improvement with added resources | High |
| Latency | Time to complete single operation | Medium |
| Capacity | Maximum concurrent load | High |

### Amdahl's Law Essentials

- Serial fraction **F** limits maximum speedup to **1/F**
- Small serialization percentages become critical at high processor counts
- Sources of serialization: shared queues, result handling, any shared state

### Lock Contention Reduction Techniques

| Technique | Description | Scalability Benefit |
|-----------|-------------|---------------------|
| Narrow scope | Hold locks briefly | Moderate |
| Lock splitting | Separate locks for independent state | Moderate |
| Lock striping | Many locks for partitioned data | High |
| Avoid hot fields | Don't cache if it creates contention | High |
| Concurrent collections | Replace synchronized wrappers | High |
| Read-write locks | Allow concurrent readers | Moderate-High |
| Atomic variables | Lock-free updates for simple state | High |

### Threading Costs

| Cost | Magnitude | Mitigation |
|------|-----------|------------|
| Context switch | 5,000–10,000 cycles | Reduce blocking, contention |
| Uncontended sync | 20–250 cycles | JVM optimizes; don't worry |
| Contended sync | Much higher | Reduce contention |
| Memory barriers | Variable | Unavoidable for visibility |

### Anti-Patterns to Avoid

❌ Premature optimization
❌ Optimizing without measuring
❌ Trading safety for performance
❌ Object pooling (in most cases)
❌ Holding locks during I/O or long computations
❌ Using synchronized wrappers instead of concurrent collections
❌ Caching values that create hot fields
❌ Assuming synchronization is "too slow"

### Performance Optimization Checklist

- [ ] Establish concrete performance requirements
- [ ] Measure current performance under realistic load
- [ ] Identify actual bottlenecks (don't guess)
- [ ] Consider scalability, not just raw speed
- [ ] Evaluate serialization sources (Amdahl's Law)
- [ ] Check CPU utilization and contention
- [ ] Minimize lock scope and granularity
- [ ] Use concurrent collections appropriately
- [ ] Measure again after changes
- [ ] Verify safety is preserved

### Key Principles

1. **Safety first** – correctness before performance
2. **Measure, don't guess** – intuition about concurrency performance is often wrong
3. **Scalability matters** – especially for server applications
4. **Serialization is the enemy** – minimize time in exclusive locks
5. **Contention is expensive** – uncontended locks are cheap, contended locks devastate performance
6. **Allocation is fast** – don't pool objects to "save" allocations


