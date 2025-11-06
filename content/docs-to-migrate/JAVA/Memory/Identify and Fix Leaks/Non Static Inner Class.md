---
weight: 600
---

# Java Memory Leaks : Non Static Inner Class

## Problem

In Java, we can create inner classes (a.k.a nested classes) as `static` and `non-static`.

- A static nested class instance can exist independently of an outer class instance.
- A non-static nested class instance is dependent on the outer class instance and implicitly holds a reference to the outer class instance.

Due to this implicit reference from a non-static nested class instance, even though the outer class instance is no longer in use, it can't be collected by the Garbage Collector.
which may lead to a memory leak.

Let's look at an example to simulate it.

```java
import java.util.ArrayList;
import java.util.List;

public class InnerOuter {

    public static void main(String[] args) throws InterruptedException {

        Outer outer = new Outer(10);
        outer = new Outer(100);
        outer = new Outer(1000);
        outer = null;

        System.gc();
        Thread.sleep(5000);

        Outer.innerList.forEach((inner) - > {
            inner.print();
        });
    }

}

class Outer {

    public static List < Inner > innerList = new ArrayList < > ();
    int val;

    public Outer(int val) {
        this.val = val;
        innerList.add(new Inner(val - 1));
        innerList.add(new Inner(val));
        innerList.add(new Inner(val + 1));
    }

    class Inner {
        int innerVal;

        Inner(int val) {
            this.innerVal = val;
        }

        void print() {
            System.out.println("Outer Val is : " + val + ", Inner Val is : " + innerVal);
        }
    }

}
```

As we saw, The static list `innerList` is holding strong references to a set of inner class instances. Due to this, the outer class instances, on which those inner class instances dependent on, cannot be collected as garbage by the Garbage Collector. This leads the system to a potential memory leak problem in Java.

## Fix

To avoid such issues, it is recommended to

- Use a static class as an inner class, as this will make sure the inner class is not dependent on the outer class instance
- If creating a strong reference to a non-static outer class, make it a Weak Reference.
- If it is a must to use `non-static` and `strongly referenced` non-static inner class, make sure it will be short-lived in memory.

Happy Coding 🙌