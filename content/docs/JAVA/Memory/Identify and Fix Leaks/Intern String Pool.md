---
weight: 800
---

# Java Memory Leaks : Intern String Pool

As we discussed before, in Java, when we defined a string value as `String s = "ABC";`. It will be stored in the **Intern String Pool** in the Heap. What happens is, if "ABC" is already in the intern string pool in the Heap, it will use the already existing "ABC". Otherwise, "ABC" will be put into the intern string pool. Simply, this pool maintains unique strings in the application, and those strings will reside in the pool until the application is terminated, because there will be **no Garbage Collection in the string pool**.

```text
Note:

When we define a String as a literal as follows,

    String s1 = "ABC";   // ABC in Intern Pool
    String s2 = "ABC";   // ABC in Intern String Pool

So,  s1 == s2 == "ABC"  is true.


But when we define a String as an Object,

    String s = new String("ABC");

This won't be interned in the pool.  
This will live as a separate object inside the heap.

    String s3 = new String("ABC");   --> Separate Object inside Heap
    String s4 = new String("ABC");   --> Another Separate Object inside Heap

So,  s1 / s2 != s3 != s4


If we want to send s3 & s4 to the intern pool, we must call the `intern()` method:

    s3 = s3.intern();   --> Put "ABC" into intern string pool (if not exists),
                           change the reference to point to the pool,
                           and the string object will be dereferenced.

    s4 = s4.intern();   --> Put "ABC" into intern string pool (if not exists),
                           change the reference to point to the pool,
                           and the string object will be dereferenced.


Now:

    s1 == s2 == s3 == s4 == "ABC"

All references point to the literal in the string pool.

```

What if we keep defining completely different, unique strings as literals?

> Each time a unique string is defined as a literal, it will be a new record in the string intern pool

```java
public static void main(String[] args) throws InterruptedException {
    while (true) {
        String userId = "USER_" + UUID.randomUUID().toString();
        System.gc(); // Intern String Pool Wont Garbage Collected 
        Thread.sleep(10);
    }
}
```

And this area of memory **does not have Garbage Collection**, and these literals exist until the application exists. So, this could lead to a **memory leak**.

So, it is recommended,

1. Use string literals: Only for predefined constants
2. Use String Objects: Dynamically created String values

Happy Coding 🙌