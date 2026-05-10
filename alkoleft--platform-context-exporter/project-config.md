---
trigger: always_on
description: Java rules for Concurrency objects
---

# Java rules for Concurrency objects

Effective Java concurrency relies on understanding thread safety fundamentals, using `java.util.concurrent` utilities, and managing thread pools with `ExecutorService`. Key practices include implementing concurrent design patterns like Producer-Consumer, leveraging `CompletableFuture` for asynchronous tasks, and ensuring thread safety through immutability and safe publication. Performance aspects like lock contention and memory consistency must be considered. Thorough testing, including stress tests and thread dump analysis, is crucial. Modern Java offers virtual threads for enhanced scalability, structured concurrency for simplified task management, and scoped values for safer thread-shared data as alternatives to thread-locals.

## Implementing These Principles

These guidelines are built upon the following core principles:

1.  **Master Thread Safety Fundamentals**: Understand and correctly apply core concepts such as synchronization (locks, conditions), atomic operations (`java.util.concurrent.atomic`), thread-safe collections (`java.util.concurrent`), immutability, and the Java Memory Model to ensure data integrity and prevent race conditions or deadlocks.
2.  **Efficient Thread Pool Management**: Utilize `ExecutorService` for robust thread management. Choose appropriate thread pool implementations and configure them with suitable sizing, keep-alive times, queue capacities, and rejection policies based on the application's workload. Implement graceful shutdown procedures.
3.  **Leverage Concurrent Design Patterns**: Implement established patterns like Producer-Consumer (using `BlockingQueue`) and Publish-Subscribe to structure concurrent applications effectively, promoting decoupling, scalability, and maintainability.
4.  **Embrace Asynchronous Programming with `CompletableFuture`**: Employ `CompletableFuture` to compose and manage asynchronous computations in a non-blocking way. Chain dependent tasks, combine results from multiple futures, and handle exceptions gracefully to build responsive and efficient applications.
5.  **Prioritize Immutability and Safe Publication**: Design classes to be immutable whenever feasible to inherently achieve thread safety. Ensure that shared mutable objects are safely published (e.g., via `volatile`, static initializers, or proper synchronization) so that their state is consistently visible to all threads.
6.  **Optimize for Performance, Considering Concurrency overheads**: Be mindful of performance implications such as lock contention (minimize scope, use finer-grained locks), memory consistency (understand happens-before, use `volatile` where appropriate), context switching overhead (size thread pools carefully), and potential issues like false sharing.
7.  **Thorough Testing and Debugging**: Rigorously test concurrent code. This includes unit tests for thread-safe components, integration tests for interactions, and stress tests to reveal race conditions or deadlocks. Utilize thread dump analysis, proper logging, and concurrency testing tools.
8.  **Adopt Modern Java Concurrency Features for Enhanced Development**: 
    *   **Virtual Threads (Project Loom)**: Embrace virtual threads via `Executors.newVirtualThreadPerTaskExecutor()` for I/O-bound tasks to dramatically increase scalability with minimal resource overhead. Avoid pooling virtual threads.
    *   **Structured Concurrency**: Use `StructuredTaskScope` to simplify the management of multiple related concurrent tasks as a single unit of work, improving error handling, cancellation, and resource management.
    *   **Scoped Values**: Prefer `ScopedValue` over `ThreadLocal` for sharing immutable data robustly and efficiently across tasks within a dynamically bounded scope, especially when working with virtual threads.

## Table of contents

- Rule 1: Thread Safety Fundamentals
- Rule 2: Thread Pool Management
- Rule 3: Concurrent Design Patterns
- Rule 4: Asynchronous Programming with CompletableFuture
- Rule 5: Thread Safety Guidelines (Immutability & Safe Publication)
- Rule 6: Performance Considerations in Concurrency
- Rule 7: Testing and Debugging Concurrent Code
- Rule 8: Embrace Virtual Threads for Enhanced Scalability
- Rule 9: Simplify Concurrent Code with Structured Concurrency
- Rule 10: Manage Thread-Shared Data with Scoped Values

## Rule 1: Thread Safety Fundamentals

Title: Understand and Apply Core Thread Safety Concepts
Description: Ensure data integrity and correct behavior in multi-threaded environments by using thread-safe data structures and appropriate synchronization mechanisms.
- Prefer `java.util.concurrent` collections over older synchronized wrappers.
- Utilize immutable objects to eliminate risks of concurrent modification.
- Employ thread-local variables for state confined to a single thread.
- Use atomic classes (`java.util.concurrent.atomic`) for lock-free operations on single variables.
- Choose flexible locking with `ReentrantLock` or `ReadWriteLock` for more complex scenarios.
- Favor `java.util.concurrent` utilities over manual `wait()/notify()`.

**Good example:**

```java
import java.util.Map;
import java.util.Queue;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ConcurrentLinkedQueue;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alkoleft/platform-context-exporter](https://github.com/alkoleft/platform-context-exporter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
