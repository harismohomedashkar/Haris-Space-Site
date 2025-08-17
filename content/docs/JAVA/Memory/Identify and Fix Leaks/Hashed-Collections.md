---
weight: 400
---

# Java Memory Leaks : Hashed Collections

Assume we are saving Java objects in hash-based collections such as `HashMap`, `HashSet`, `HashTable`, etc. These collections will handle the data structure completely based on the similarity of the hash value of that object.

## Problem

However, in Java, by default, 2 objects with the same values will have 2 different hash codes, since the default hash value calculation is not based on the field values. Due to this, even if we put 2 objects with identical field values into Hash-based collections, it will be considered as two different components. 

The result will be that the size of the data structure will grow unnecessarily, since two objects with identical values will still be considered as 2 components. This can become a source of memory leaks in the system.

```java
public class HashBasedLeak {
    public static void main(String[] args) throws InterruptedException {
        Thread.sleep(10000);
        HashMap < Item, Integer > ItemInStock = new HashMap < > ();
        for (int i = 0; i < 1000000; i++) {
            ItemInStock.put(new Item("T-Shirt", "Blue", 19.99), i);
            ItemInStock.put(new Item("Sneakers", "White", 49.99), i);
            ItemInStock.put(new Item("Backpack", "Black", 29.99), i);
            if (i % 10000 == 0) {
                Thread.sleep(100);
                System.gc();
            }
        }
        System.out.println("Items in Shop Count : " + ItemInStock.size());
    }
}

class Item {
    private String name;
    private String color;
    private double price;
    public Item(String name, String color, double price) {
        this.name = name;
        this.color = color;
        this.price = price;
    }
}
```

![](/images/hash-001.png)

## Fix

As shown above, even though there are 3 identical items in the context of stock. The map records `3000000`. To solve this issue, we need to override the `equals` and `hashCode` functions as below.

> RULE OF THUMB : IF TWO OBJECTS EQUALS, THEN HASH CODES ALSO SHOULD BE EQUAL

```java
import java.util.Objects;
public class Item {
    private String name;
    private String color;
    private double price;
    public Item(String name, String color, double price) {
        this.name = name;
        this.color = color;
        this.price = price;
    }
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Item item = (Item) o;
        return Double.compare(item.price, price) == 0 &&
            Objects.equals(name, item.name) &&
            Objects.equals(color, item.color);
    }
    @Override
    public int hashCode() {
        return Objects.hash(name, color, price);
    }
}
```

Now let's try again,

![](/images/hash-002.png)

As we can see, there is a huge difference in memory and application results between the approaches. Hence, hash-based collections should be used carefully to prevent memory leaks as well as logical issues in the application.

Happy Coding 🙌