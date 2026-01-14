Chapter 1 – Introduction to Concurrency

Writing correct concurrent programs is significantly harder than writing sequential ones because many more things can go wrong due to unpredictable interactions between threads. Despite this complexity, concurrency is essential in Java: threads simplify asynchronous programming, improve system responsiveness, and allow applications to fully utilize modern multiprocessor hardware.

Why Concurrency Exists

Concurrency emerged to address three main problems:

Resource utilization: While one task waits (e.g., for I/O), others can run.

Fairness: Multiple users or programs can share CPU time.

Convenience: Complex systems can be decomposed into cooperating tasks.

Early systems used processes, which are isolated programs with limited communication mechanisms. Threads evolved as a lighter-weight alternative: multiple execution paths within a single process that share memory and resources.

Threads and the Sequential Model

Most programming languages, including Java, follow a sequential model that is intuitive and human-friendly. Threads preserve this mental model by allowing developers to write mostly sequential logic while still handling asynchronous events. However, because threads execute concurrently and share memory, incorrect coordination can lead to subtle and dangerous bugs.

Benefits of Threads

Threads provide several major advantages:

Exploiting multiple processors: Multithreaded programs can run in parallel on multiple CPUs, dramatically improving throughput.

Simpler modeling: Assigning threads to tasks creates the illusion of sequential execution and reduces complexity.

Handling asynchronous events: Blocking I/O becomes manageable when each task has its own thread.

Responsive user interfaces: Long-running tasks can run in background threads, keeping the UI responsive.

Threads are foundational to many Java frameworks (servlets, RMI, Swing, garbage collection), making concurrency unavoidable in real-world applications.

Risks of Threads

Concurrency introduces three major categories of risk:

1. Safety Hazards

Without proper synchronization, threads may interfere with each other, leading to race conditions. Even simple operations (like value++) are not atomic. Java provides synchronization mechanisms to ensure predictable behavior, but developers must use them correctly.

2. Liveness Hazards

Threads can get stuck indefinitely, leading to failures such as:

Deadlock

Starvation

Livelock

These bugs are often timing-dependent and difficult to reproduce.

3. Performance Hazards

Threads introduce overhead:

Context switching

Synchronization costs

Cache invalidation

Poorly designed concurrency can degrade performance rather than improve it.

Threads Are Everywhere

Even if an application does not explicitly create threads, frameworks do. Components invoked by:

Timers

Servlets/JSPs

RMI

Swing/AWT

are all executed in framework-managed threads. This makes thread safety contagious: once shared state is accessed by multiple threads, all related code must be thread-safe.

Core Message

Concurrency is powerful but dangerous. Because threads are pervasive in Java, developers must understand concurrency fundamentals to write safe, responsive, and scalable applications. Proper synchronization, careful design, and awareness of concurrency risks are essential.
