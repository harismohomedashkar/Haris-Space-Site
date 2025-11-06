# Tackling The Multi-Threaded Environments

![](/images/multi-thread.png)

## Introduction

A multi-threaded environment is a computing environment where multiple threads try to use the same shared resources to achieve their desired goals. However, unexpected issues can occur when these threads access shared resources concurrently. In order to develop robust, error-free applications, we need to tackle these issues effectively. Here we discuss most common known issues we need to tackle in a multi-threaded environment.

## Deadlock

### Problem

A deadlock occurs when two more threads are waiting for each others to release resources that they needs to complete their tasks. Since non of the threads cannot complete their tasks without acquiring desired resources and everyone is waiting for each other to release them, the threads will be locked indefinitely. 

### Example 

For example let's say **Thread 1 need *Res1, Res5, Res2* in Given Order** and **Thread 2 need *Res2, Rest1, Res7* in Given Order**.

![](/images/deadlock-1.png)

*Thread 1* --> First Acquires Resource 1, 5 and Waiting Thread 2 to release Resource 2 so it can complete task and release Resource 2, 5, 1

*Thread 2* --> First Acquires Resource 2 and Waiting Thread 1 to release Reousrce 1 so it can complete task and release Resource 7, 1, 2

Since both waiting for each other to release resources, both stuck indefinitely.

### How to solve

1. The eaiset way to solve this issue is, introducing a locking order to avoid circular locking, for example we can say each thread aquire resources according to Res No Value `Low` => `High` in that order.

![](/images/deadlock-2.png)

> (*) => If Thread 1 Already acquired Resource 1, Thread 2 will need to wait till Thread 1 releases Resource 1 to continue, but not stuck indifinitely as was before.

2. Another approach to solve this issue is, give each lock timeouts to encourage threads to giveup when time is up, hence resources will not be locked indifinitely.


## LiveLock

### Problem 

A livelock is a situation where two or more processes are constantly changing their state in response to each other, but none of them can make any progress. 

> This does not mean they are stuck. Livelocks means they're busy but not productive.

### Example

Let's say we defined two workers to process a queue. When queue have new messages, 

- One of the worker should remove that message
- Mark itself as isBusy = true;
- Start processing that message. (this takes 100ms)
- Mark itself as isBusy = false;

Both workers are initially initialized with `isBusy = false`.

```text
0ms   : Worker 1 initialized (isBusy = false)
10ms  : Worker 2 initialized (isBusy = false)
110ms : A new message arrives in the queue
120ms : Worker 1 checks Worker 2 (sees not busy), so it waits politely
122ms : Worker 2 checks Worker 1 (sees not busy), so it also waits politely
130ms : Both workers check again, see each others not busy, and keep backing off
```

Each worker sees the other as available and backs off, **trying to be polite or avoid race conditions**.
Eventually no one process the message.

### Fixing

- ***Assign Priority*** : We can configure worker one always take messages and process. And configure Worker 2 to process messages only when worker one is found busy.

- ***Random Backoff*** :  We can introduce some random probability, like if worker 2 found worker 1 free, 80% random chance worker 2 backs off, and 20% random chance worker 2 also trying to involve.

- ***TimeOut*** : If found, the message existing in the queue for more than 5 sec, worker should try to process it, regardless of the other worker's state.

## Starvation

### Problem

Starvation occurs when a thread is indefinitely blocked from accessing shared resources. Eventhough those resources become available for sometime, 
it quickly become unavailable before starving thread get chance to lock the resource,
due to other threads get higher chances on accessing those resources.

### Cause and Fix
- ***Priority*** : One Lower Priority thread have to starve, when Large Amount Higher Priority threads available. 
> ***Fix*** : Use ***priority aging*** to increase the priority of waiting threads with time.

- ***Unfair Scheduling Policies*** : Some scheduling algorighms may give higher chances to specific type of threads.
> ***Fix*** : Use a fair scheduler like ***round-robin***.

- ***Long Running Tasks*** : If task runs for a long time and uses some resources, others threads may be blocked from accessing those resources.
> ***Fix*** : Break long tasks into ***smaller chunks*** or use ***timeouts*** for resource access.

- ***Synchronized Blocks*** : If thread entered a synchronized code block, which executes for long time, all other threads have to wait.
> ***Fix*** : make sure ***synchronized block*** is added for ***necessary logic*** only.


## Race Condition

### Problem

Race Condition occurs when two or more threads trying to update same shared resources at same time leads to data inconsistency.


### Example 

![](/images/race-condition.png)

```text
Now our expected result is x = 10 + 1 + 2 which is 13
But the value we got is x = 12.
```

This incident we refer as ***Race Condition***.

### Fix

- Make sure we update x within `synchronized` block
- Use `AtomicIntegeer` for x 
- Use Java Lock (Reentrant Lock)

Yeah those are the some of commonly known issues we need to tackle in multi threaded environemnts. Hope this blog post was helpful.

Thank You.\
Happy Coding 🙌