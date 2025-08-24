---
weight: 900
---

# Java Memory Leaks : Thread Local

## Introduction

Now let's see how thread locals can cause memory leaks in Java. For this, first we need to know how thread locals work internally. 

Each thread has its own `ThreadLocalMap`.

```
Thread1 : Map<WeakReference<ThreadLocal<?>>, T> threadlocalmap = {}
Thread2 : Map<WeakReference<ThreadLocal<?>>, T> threadlocalmap = {}
Thread3 : Map<WeakReference<ThreadLocal<?>>, T> threadlocalmap = {}
Thread4 : Map<WeakReference<ThreadLocal<?>>, T> threadlocalmap = {}
```

To understand memory leaks, let's consider the following example:

```java
public class ThreadLocalLeakExample {
    public static void main(String[] args) throws Exception {
        Thread.sleep(10000);
        ExecutorService executor = Executors.newFixedThreadPool(4);
        // Submitting Tasks By Reusing 4 Threads Repeatedly
        for (int i = 0; i < 1000; i++) {
            executor.submit(() - > {
                ThreadLocal < byte[] > threadLocal = new ThreadLocal < > ();
                threadLocal.set(new byte[10 * 1024 * 1024]);
                System.out.println("ThreadLocal set in " + Thread.currentThread().getName());
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }
        executor.awaitTermination(10000, TimeUnit.MILLISECONDS);
        System.gc();
        Thread.sleep(10000);
        System.out.println("\nTerminating...");
        executor.shutdown();
    }
}
```

![](/images/thread_local_1.png)

In the above code, we have created a thread pool with 4 threads and are reusing it again and again 1000 times. Each time we reuse the thread in the pool, we are creating a ThreadLocal variable for that scope and setting a value using `threadLocal.set(new byte[10 * 1024 * 1024]);`. While doing this 1000 times reusing 4 threads, we can see a rise in memory usage because each time a task is executed by a thread, it creates a new mapping for ThreadLocal → Byte in the ThreadLocalMap, and this takes memory.

Then we're waiting for all tasks to be completed and signaling the garbage collector, which may run after this. However, we can see there is still a considerable amount of memory used even after the garbage collection. 

## Problem

Let's see why this is happening. After execution of all submissions, the ThreadLocal objects in the heap won't have any references. So these will be collected by the GC. This is because, even if they are referred to as keys in the ThreadLocal map, they are marked as **weak references**, so the ThreadLocal objects representing keys in the map won’t prevent them from being garbage collected.

Next, in addition to ThreadLocal, each thread’s ThreadLocalMap will have Byte objects as values for each ThreadLocal created. On average, each ThreadLocalMap will have ~250 MB objects (since we ran 10 B times 1000 using 4 threads), and these are marked as **strong references** in ThreadLocalMap. Even though they don’t have any references, these won't be collected by the GC.

So there will be mappings like below in Thread Local Map:

> `null` (because ThreadLocal is weak reference and is collected by GC) --> `Byte` (strong reference, won’t be collected by GC even though it cannot be accessed through the key, which is null)

This leads the system into a memory leak. Assume we use a server like **Tomcat**, which uses a thread pool and reuses threads to handle incoming requests—this type of leak may slowly kill the program by consuming memory over time.

## Solution

As a solution, it is highly recommended to use `threadLocal.remove()` immediately after the relevant ThreadLocal variable is used, before the termination of the task. So in our example, we can fix this issue as follows:

```java
public class ThreadLocalLeakExample {
    public static void main(String[] args) throws Exception {
        Thread.sleep(10000);
        ExecutorService executor = Executors.newFixedThreadPool(4);
        // Submitting Tasks By Reusing 4 Threads Repeatedly
        for (int i = 0; i < 1000; i++) {
            executor.submit(() - > {
                ThreadLocal < byte[] > threadLocal = new ThreadLocal < > ();
                threadLocal.set(new byte[10 * 1024 * 1024]);
                System.out.println("ThreadLocal set in " + Thread.currentThread().getName());
                threadLocal.remove(); // <-- Remove ThreadLocal value from ThreadLocalMap after usage
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }
        executor.awaitTermination(10000, TimeUnit.MILLISECONDS);
        System.gc();
        Thread.sleep(10000);
        System.out.println("\nTerminating...");
        executor.shutdown();
    }
}
```

Now we have added `threadLocal.remove()`; to clear the ThreadLocalMap from unused mappings. After execution, we're signaling the garbage collector, and the result looks as follows:

![](/images/thread_local_2.png)

As we can see, just adding a single line of code to properly clean the ThreadLocalMap saved a huge amount of memory and prevented the application from a memory leak. 

> It is advised to use `.remove()` inside a finally block (so this will be executed even if some error is caught)

Clear the ThreadLocal variable after usage to prevent memory leaks, as shown below:

```java
try {
    User user = extractUserData(request);
    UserUtils.setThreadLocalUser(user);
} catch (Exception ex) {
    // We can handle exception here
} finally {
    UserUtils.clearThreadLocal();
}

class UserUtils {
    private static ThreadLocal < User > threadLocalUser = new ThreadLocal < > ();
    public static void setThreadLocalUser(User user) {
        threadLocalUser.set(user);
    }
    public static User getThreadLocalUser() {
        return threadLocalUser.get();
    }
    public static void clearThreadLocal() {
        threadLocalUser.remove();
    }
}
```

In summary, ThreadLocal is a powerful feature in Java, but it should be used carefully.

Happy Coding 🙌