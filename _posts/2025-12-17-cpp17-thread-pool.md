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

- ## `std::thread`

  `std::thread` is the fundamental class for creating and managing threads in C++. 
  It is introduced in C++11 and continues to be a core component in C++17.
  With `std::thread`, you can create a new thread by passing a callable object

  ```cpp
  #include <thread>
  #include <iostream>
  #include <chrono>

  void workerFunction() {
      std::cout << "Worker thread is running." << std::endl;

      // Simulate some work
      std::this_thread::sleep_for(std::chrono::seconds(1));
  }

  int main() {
      std::thread worker(workerFunction);
      worker.join(); // Wait for the worker thread to finish
      return 0;
  }
  ```

  Upon creation, the thread starts executing the provided function concurrently with the main thread.`std::thread::join()` is used to block the calling thread until the thread represented by the `std::thread` object has completed its execution. To check if a thread is joinable, you can use the `std::thread::joinable()` method, which returns `true` if the thread is joinable and `false` otherwise.

  ```plaintext

  Main Thread
      |
      |-- Create std::thread --> Worker Thread
      |                          |
      |                          |-- Execute workerFunction()
      |                          |
      |                          |-- Finish execution
      |
      |-- Call join() ---------> Wait for Worker Thread to finish
      |
      |-- Continue execution after Worker Thread finishes

  ```

  The worker function can be of multiple types, including: 
    - Free functions (eg. `void workerFunction()`)
    - Lambda expressions (eg. `[]() { /* ... */ }`)
    - Functor objects (eg. `struct Worker { void operator()() { /* ... */ } };`)
    - Member functions (eg. `class MyClass { void memberFunction() { /* ... */ } };`)
  The usage of different callable types is different:
  
  ```cpp

  // Using a lambda expression

  std::thread worker([]() {
      std::cout << "Worker thread is running." << std::endl;
  });

  ```

  ```cpp

  // Using a functor object

  struct Worker {
      void operator()() {
          std::cout << "Worker thread is running." << std::endl;
      }
  };

  std::thread worker(Worker());

  ```

  ```cpp

  // Using a member function

  class MyClass {
  public:
      void memberFunction() {
          std::cout << "Worker thread is running." << std::endl;
      }
  };

  MyClass obj;
  std::thread worker(&MyClass::memberFunction, &obj);

  ```

  The callable object can also accept parameters, which can be passed to the thread constructor:

  ```cpp

  void workerFunction(int id) {
      std::cout << "Worker thread " << id << " is running." << std::endl;
  }

  std::thread worker(workerFunction, 1); // Pass 1 as the parameter

  ```

  One thing to notice is that the return type of the callable object must be `void`. If the callable object has a non-void return type, it will result in errors depending on the compiler. To handle non-void return types, we can use `std::packaged_task`,
  `std::promise`, and `std::future`, which will be discussed later in this article.

  The `std::thread` class is not copyable, meaning you cannot copy a `std::thread` object. This is to prevent multiple threads from trying to manage the same thread of execution, which could lead to undefined behavior.
  However, `std::thread` is movable, allowing you to transfer ownership of a thread from one `std::thread` object to another using move semantics.

- ## `std::mutex`  and  `std::lock_guard`

  `std::mutex` is a synchronization primitive that can be used to protect shared data from being accessed simultaneously by multiple threads. It works by allowing only one thread to lock the mutex at a time, preventing other threads from accessing the protected data until the mutex is unlocked. `std::lock_guard` is a convenient RAII-style wrapper for `std::mutex` that automatically locks the mutex when the `std::lock_guard` object is created and unlocks it when the object goes out of scope.

  If multiple threads access shared data without proper synchronization, it can lead to data races and inconsistent results. 
  A diagram is shown below:

  ```plaintext
    Thread 1                     Thread 2
      |                             |
      |-- Read sharedResource       |
      |                             | -- Read sharedResource (The value is the same as Thread 1)
      |-- Increment sharedResource  |
      |                             | -- Increment sharedResource
      |-- Write sharedResource      |
      |                             | -- Write sharedResource
    Final value of sharedResource may be inconsistent due to race conditions.
  ```

  A simple example of using `std::mutex` and `std::lock_guard` is shown below:


  ```cpp
  #include <iostream>
  #include <thread>
  #include <mutex>
  #include <vector>

  std::mutex mtx; // Mutex for synchronizing access to shared resource
  int sharedResource = 0; // Shared resource

  void workerFunction(int id) {
      // Lock the mutex using std::lock_guard
      std::lock_guard<std::mutex> lock(mtx);

      // Critical section: safely access and modify the shared resource
      ++sharedResource;
      std::cout << "Worker " << id << " incremented sharedResource to " << sharedResource << std::endl;
  }

  int main() {
      const int numWorkers = 5;
      std::vector<std::thread> workers(0);

      // Create and start worker threads
      for (int i = 0; i < numWorkers; ++i) {
          workers.emplace_back(workerFunction, i);
      }

      // Wait for all worker threads to finish
      for (auto& worker : workers) {
          worker.join();
      }

      // Final value of sharedResource
      std::cout << "Final value of sharedResource: " << sharedResource << std::endl;

      // If mutex is not used, the output may be inconsistent due to
      // race conditions.

      return 0;
  }
  ```

  There are other lock guards available in C++17, such as `std::unique_lock` and `std::shared_lock`, which provide more flexibility and features for managing locks. While `std::lock_guard` is a simple and efficient way to manage mutex locks, `std::unique_lock` allows for more complex locking strategies, such as deferred locking and timed locking. `std::shared_lock` is used for shared (read) access to a resource, allowing multiple threads to read the resource simultaneously while preventing write access.
  When in need of lock two mutexes, dead lock may occur if two threads try to lock the same mutexes in different orders.
  The following is a diagram:

  ```plaintext
    Thread 1                     Thread 2
      |                             |
      |-- lock(mutex1)              |
      |                             |-- lock(mutex2)
      |                             |
      |-- lock(mutex2)              |
      |                             |-- lock(mutex1)
      |                             |
    Deadlock occurs here as both threads are waiting for each other to release the mutexes.

  ```

  To avoid deadlock when locking multiple mutexes, two approaches can be used: `std::lock` and `std::scoped_lock`.
  `std::lock` is firstly introduced in C++11, which locks multiple mutexes without the risk of deadlock.
  `std::scoped_lock` is introduced in C++17, which is a variadic template that can lock multiple mutexes in a deadlock-free manner.

  ```cpp

  // Locking two mutexes without deadlock

  std::mutex mutex1;
  std::mutex mutex2;

  std::lock(mutex1, mutex2); // Lock both mutexes without deadlock
  std::lock_guard<std::mutex> lock1(mutex1, std::adopt_lock);
  std::lock_guard<std::mutex> lock2(mutex2, std::adopt_lock);

  ```

  ```cpp

  // Locking two mutexes with std::scoped_lock (C++17)

  std::mutex mutex1;
  std::mutex mutex2;

  std::scoped_lock lock(mutex1, mutex2); // Lock both mutexes without deadlock

  ```

- ## `std::condition_variable`

- ## `std::future` and `std::promise`

- ## `std::packaged_task`

---

# Implementation of Thread Pool

---

[^book_1]: Gamma, E., Helm, R., Johnson, R., & Vlissides, J., 1994, Design Patterns: Elements of Reusable Object-Oriented Software. Addison-Wesley Professional.








