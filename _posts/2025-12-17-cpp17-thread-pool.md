---
title: '[C++17] Thread pool'
date: 2025-12-17
permalink: /posts/2025/12/cpp17-thread-pool/
tags:
  - C++17
  - Multi-thread
  - astroDEM
---

<font size="3">
This article is part of the <b>astroDEM</b> development diary. <br>
It describes the implementation of a thread pool in C++17 for parallel processing tasks in the <b>astroDEM</b> project. <br>
</font>

---

\
<b>C++17</b> introduced several features that make it easier to implement a thread pool. 
A thread pool is a collection of worker threads that can be used to execute tasks concurrently, 
improving performance for CPU-bound operations.
\
\
In `astroDEM`, the thread pool is mainly used for parallel calculations of forces, including gravitational forces 
and contact forces between grains. It is designed as part of the `astroDEM::Simulation::ParallelToolKit` module.

Though there are existing thread pool libraries available,
we decided to implement our own thread pool in `astroDEM` using C++17 standard for 
the following reasons:
  - Customization: Implementing our own thread pool allows us to tailor it to the specific needs of `astroDEM`, optimizing for the types of tasks and workloads we encounter.
  - Control: Having our own implementation gives us full control over the behavior and performance characteristics of the thread pool, allowing us to fine-tune it as needed.
  - C++17 features: Leveraging C++17 features allows us to create a modern and efficient thread pool that takes advantage of the latest language capabilities.
      - We didn't choose a higher standard (like C++20 or C++23) for:
           - maintain compatibility with a wider range of compilers and systems.
           - ensure stability and reliability by avoiding potential issues with newer, less-tested features.
  - Learning experience: Implementing a thread pool from scratch provides valuable insights into concurrent programming and thread management in C++.

---

# Quick links

- [The Story of The Thread Pool](#the-story-of-the-thread-pool)
- [C++17 Threading Features](#c17-threading-features)
- [Implementation of Thread Pool](#implementation-of-thread-pool)

---

# The Story of The Thread Pool 

Before the story of the thread pool, let's briefly review some basic concepts.


Process and thread are fundamental units of execution in an operating system.
A process is an independent program that runs in its own memory space, while a thread is a smaller unit of execution within a process that shares the same memory space with other threads in the same process.
Threads are used to achieve parallelism and improve performance, especially for CPU-bound tasks. Threads within the same process have the below advantages:
- Shared memory space: Threads can access the same memory space, making it easier to share data between them without the need for inter-process communication (IPC) mechanisms.
- Lower overhead: Creating and managing threads is generally more lightweight than processes, leading to lower overhead
- Faster context switching: Switching between threads is typically faster than switching between processes, resulting in better performance for multi-threaded applications.
- Process Dependence: Threads are dependent on the process they belong to. If the process terminates, all its threads are also terminated.

The thread pool is a design pattern that involves creating a pool of worker threads that can be reused to execute multiple tasks concurrently.
This approach helps to reduce the overhead of creating and destroying threads for each task, leading to improved performance.

- ## Ancient times: processes and multitasking (1970s - 1980s)

  Concurrent programming has been around since the early days of computing.
  Early operating systems used cooperative multitasking, where processes voluntarily yielded control to allow other processes to run. Every process had its independent memory space, which provides good isolation but makes inter-process communication more complex. 

  The expense of creating and managing processes (switching, memory allocation, etc.) is 
  high, leading to inefficiencies in resource utilization. When dealing with multiple 
  instantaneous tasks, the overhead of process management can become a bottleneck.
  At this time, the concept of threads was introduced to allow multiple execution paths within a single process, sharing the same memory space.

```cpp
// An example of processes and multitasking will be added here.
```

- ## Early developments: the rise of threads (1980s - 1990s)

  Thread was firstly implemented in operating systems like Solaris, POSIX, and Windows NT.
  Threads allowed multiple execution paths within a single process, sharing the same memory space. This made it easier to share data between threads and reduced the overhead of context switching compared to processes.

  However, new challenges arose:
  - Uncontrolled thread creation: Without proper management, applications could create too many threads, leading to resource exhaustion and performance degradation.
  - Synchronization issues: With multiple threads accessing shared resources, synchronization mechanisms (like mutexes and semaphores) became necessary to prevent data races and ensure consistency.
  - Managing thread lifecycle: Properly managing the creation, execution, and termination of threads became crucial to avoid resource leaks and ensure efficient resource utilization.

  The need for a more structured approach to thread management led to the development of the thread pool concept.

- ## Concept and early implementations (late 1990s)

  The thread pool concept emerged as a solution to the overhead associated with creating and destroying threads for each task and to manage the lifecycle of threads more effectively.
  Similar concept was introduced in "Design Patterns: Elements of Reusable Object-Oriented Software"[^book_1], though not specifically called a "thread pool".

  Some early implementations of thread pools including:
   - Jave web servers: Early web servers like Apache Tomcat implemented thread pools to handle incoming HTTP requests efficiently.
   - Database connection pools: Database systems like Oracle and MySQL used thread pools to manage database connections and execute queries concurrently.

  Core components of a thread pool is established at this time:
    - Worker threads: A fixed number of threads that are created at the start and reused to
      execute tasks.
    - Task queue: A queue that holds tasks waiting to be executed by worker threads.
    - Controller: A component that manages the lifecycle of worker threads and the task queue.

- ## Modern implementations (2000s - present)

  In 2004, Java 1.5 introduced the `java.util.concurrent` package, which included a built-in thread pool implementation called `ExecutorService`. 
  This provided a standardized way to create and manage thread pools in Java applications.

  In 2005, .NET made a quick follow-up with the introduction of the `ThreadPool` class in the `System.Threading` namespace, providing similar functionality for C# and other .NET languages.

  In C++, the Standard Template Library (STL) introduced threading support in C++11, including features like `std::thread`, `std::mutex`, and `std::condition_variable`.
  However, C++11 did not include a built-in thread pool implementation. 
  
  In current days, multi-threading and thread pools are widely used in various programming languages and frameworks. They are also facing more challenges and more complex 
  algorithms are developed to improve their efficiency and scalability, such as work-stealing algorithms and reactive programming models. Some of the popular thread pool libraries include:
   - Java's `ExecutorService`
   - .NET's `ThreadPool`
   - C++ libraries like `Boost.Asio` and Intel Threading Building Blocks (TBB)

---

# C++17 Threading Features

C++17 introduced several features that enhance the threading capabilities of the language, making it easier to implement concurrent programming patterns like thread pools.
In this section, we will explore some of the key C++17 threading features that are relevant to our thread pool implementation in `astroDEM`.

---

# Implementation of Thread Pool

---

[^book_1]: Gamma, E., Helm, R., Johnson, R., & Vlissides, J., 1994, Design Patterns: Elements of Reusable Object-Oriented Software. Addison-Wesley Professional.








