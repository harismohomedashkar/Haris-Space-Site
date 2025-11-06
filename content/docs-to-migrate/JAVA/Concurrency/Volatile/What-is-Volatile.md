---
weight: 100
---
# Java : What is Volatile ?

![](/images/Screenshot%202025-01-29%20111128.png)

## Introduction

Let's assume we have a *TicketBook* class. Assume we maintain the number of remaining tickets in a shared counter variable named *availableTickets*. See the code below:
```java
class TicketBook {

  private static int availableTickets = 1234436;

}
```

Now let's assume the class is a singleton class. So only one instance of the *TicketBook* object exists during the whole runtime and all users using that instance.

Assume 2 users came to use this *TicketBook* object in two different threads on two different CPUs. By default behavior of CPU caching, each thread may have its own local copy of the *availableTickets* variable, as shown below:

![](/images/AVvXsEhUTOi4ZWo0ljfXnvjnuwBbweCBIn7krEUslTmnIbfGrPwL2FgCQw-LzFUqbfG0Vv24JcbzpoEU4OsRZG4avmqad04uEWX6wx5cVM5K5l-FwlcpDNafTjJsEFFc9ZNh2Kr27dX4b7_8l3nIg8JZ2IXK5b0cxhN1FFdK70lxdGJDSQXAHCTcIqi6AowyjmrO)

```text
Main Memory:
availableTickets = 1234436

Thread 1 (User 1):
availableTickets = 1234436

Thread 2 (User 2):
availableTickets = 1234436
```
## Problem

Now let's say *User 1* buys 10 tickets. First, the system will update those values in the local cache of the CPU. However, the change may not reflect in the original variable copy in the main memory instantly. 
*(even though modern hardware addresses this issue and provides robust cache coherence techniques, it's not a good practice to rely on hardware fixes\)*. Now the values are as follows, which leads to data inconsistency:

```text
Main Memory:
availableTickets = 1234436

Thread 1 (User 1):
availableTickets = 1234426

Thread 2 (User 2):
availableTickets = 1234436
```
## Solution

As of now we understand the issue is, the time that is taken to reflect the cache change from CPU cache to the main memory variable, and then time that is taken to reflect that change from main memory to other threads. Which is leading to a data inconsistency between threads.
  
As a solution to this problem, what if we can ask CPU to *not to cache that specific variable*. Instead, directly read / write from main memory. Yes, that is where **volatile keyword** comes into play. 

## Understanding Volatile

As discussed, the volatile keyword ensures that a variable is never cached and is always read and written directly from / to main memory. This keyword cannot be used with classes or methods. It can only be used with variables.
Essentially, it guarantees visibility that all threads see the same memory location where the variable is stored in main memory, which prevents data inconsistency between threads.

## When to Use Volatile

Volatile should be used when we have a shared variable that is referred to and updated by multiple threads. It ensures that all threads will see the updated value of the volatile variable after one thread changes it.
If we do not use a volatile variable, the compiler can reorder the code and may write to the cache value of the variable instead of reading from the main memory. Hence, it also prevents the compiler from reordering code.

```text
Note: When a variable is not shared between multiple threads, we do not need to use the volatile keyword with that variable. Using volatile unnecessarily introduces performance issues by avoiding caching at the thread level.
```

## Resolving Some Confusions

### Static vs Volatile

- ***Static***: Static means all the objects created from that class refer to a common variable defined in memory. Each object does not get a chance to initiate its own variable. But this does not mean these static variables cannot be cached at the CPU level.
- ***Volatile***: This keyword ensures that variables cannot be cached away from main memory.

### Synchronized vs Volatile

- ***Synchronized***: It restricts the execution of a code block until a thread acquires the necessary lock.
- ***Volatile***: This deals with memory caching and ensures visibility of changes across threads.

Next, let's understand the volatile keyword with a coding example : [Java : Volatile Coding Example](/docs/java/concurrency/volatile/coding-example/)

Happy Coding 🙌
