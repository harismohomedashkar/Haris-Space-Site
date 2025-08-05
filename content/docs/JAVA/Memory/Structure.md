---
weight: 100
---

# JVM: Memory Structure

## Introduction

Before diving deep into memory leaks, memory debug, garbage collection, etc. Let us first focus on learning about how the Java Virtual Machine (JVM) structures memory into different spaces to handle programs efficiently.

In a classic Java application, it uses the following types of memory during the execution.

- Stack
- Meta Space
- Heap

We can put these things into an image as follows,

![](/images/JavaMemory.png)


## Stack

Stack memories are managed by Threads. Each thread has its own stack memory. When a thread is getting destroyed, the stack of that thread will also be removed. Thread stores local variables, return addresses, arguments, etc., in the stack.

However, the stack of `Platform Thread` and `Virtual Thread` is handled differently.

###	Platform Thread Stack 

- Location: Native Memory
- Size: Fixed 
- Size Configuration:
    - CMD Args : Can configure through `-Xss` => `-Xss512k` means 512 KB and `-Xss1m` means 1 MB fixed sizes
	- Programically : Configure by `Thread thread = new Thread(null, task, "MyStack", 512 * 1024);` ==> 512 KB
- Creation : Can be created by using `new Thread(...)`

### Virtual Thread Stack 

- Location: JVM Heap
- Size Configuration: Size changes dynamically, can't set beforehand
- Can be created by using `Thread.startVirtualThread(...)`

Also, if the stack gets out of space, it will throw a `StackOverflow` error. This may be due to setting a lower value for stack size. But if we increase the stack size, it will result in higher memory usage. So, the size **must be tuned carefully** after analyzing the application resource usage.

![](/images/StackAndHeap.png)

Below I have noted down few points that we need to understand about, how data is spread across these regions.

1.	Primitive variables are stored where they're declared. 
    - If declared in Stack -> Stack
    - If declared as Object Field -> Within Object Field Inside Heap
2.	Non-Primitive: Created in Heap, and reference will be saved at the place where it's declared (Stack or Object Field in Heap) 
    - String s = "aaa" -> String Constant Pool in Heap
    - new String("aaa") -> New Object in Heap
    - new Object() -> new Object in Heap

> Variables declared within a method (even if it's in Object) will be loaded into the relevant thread stack (in the stack that can store value or reference based on data type), since those are local variables to the thread.

## Meta Space

Meta space, introduced in `Java 8`, replaces `Permanent Generation`. Unlike `Permanent Generation`, Meta Space can grow dynamically, and it's allocated out of native memory and managed by Native Memory. 

In a Java program, class definitions, bytecode, static variables, constant pool, interfaces, etc., go into Meta Space. Unlike Stack, the whole JVM instance will have only one Meta Space.

Since this area is shrinking and growing dynamically, for better resource management, we can tune the area using the following parameters,

```text
-XX: MetaspaceSize initial size
-XX: MaxMetaspaceSize=3200m Tune: Max size
-XX: MinMetaspaceFreeRatio — Specifies the minimum percentage of reserved memory after garbage collection.
-XX: MaxMetaspaceFreeRatio — Specifies the maximum percentage of reserved memory after garbage collection.
```

> However, if the usage exceeded maximum Meta Space Size allocated, then a `java.lang.OutOfMemoryError: Metaspace` exception will be thrown.


## Heap

The entire JVM instance will have only one Heap, and it will be created during the start of the JVM. The Heap is shared across all threads and stores the Objects and Arrays.

![](/images/Heap.png)

The variables in the stack will access these Objects and Arrays using reference pointers stored in the Stack. However, along with time, there can be references that can no longer be used. So, Objects that belong to these references become unreachable. Garbage Collector primarily works in this area to reclaim memory by removing these unused objects.

In JVM, Heap can grow/shrink dynamically within default/configured limits.

```text
-Xms: min heap size
-Xmx: max heap size
```

**The Heap can be divided into 2 Areas.**

### Young Generation
- Relatively new objects get stored here
- Can be divided into 3 subspaces\
    -> Eden\
    -> S0 Survivor\
    -> S1 Survivor

Initially, Eden is getting filled, then minor GCC will run, when Eden got full, as shown below,

```text

from_space = S0
to_space = S1

While True:

    If Eden is full: // trigger minor gcc

        move_survivors(eden + from_space) → to_space
        clear(eden)
        clear(from_space)
        swapes roles(from_space, to_space) 
        // if now S1 is "from" and S2 is "to" --> then S1 becomes "to" and S2 becomes "from"  

        If long_lived_objects_exist || survivor spaces filled || Object too Large:
            promote_to_old_gen()

```

Space Configuration
```text
-Xmn: young gen fixed size
-XX: NewSize: Youn gen min
-XX: MaxNewSize: Young gen max size
```

### Old Generation (Tenured)
- Major GCC will be run in this region
- Old objects which survived several GCC (can configure limit by user) will be moved into this area.

Likewise JVM uses robust memory struture to handle the program efficiently within a give memory contraints. Here we have discussed only basic level of details of the 
JVM memory structure. Next we will discuss how Java memory leak can happen and what are steps we can take to prevent it.

Happy Coding 🙌



