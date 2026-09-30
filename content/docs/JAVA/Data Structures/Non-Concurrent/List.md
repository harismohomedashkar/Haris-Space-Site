---
#786
weight : 10
---

# Java: Lists

In this section, we will discuss the commonly used Java list data types, how these structures work under the hood, the advantages and disadvantages of each, and provide a comprehensive summary of each.

## ArrayList

First, we will be focusing on ArrayList, the most generally used list type in Java. For more details about the functions it supports, please refer to [documentation](https://docs.oracle.com/javase/8/docs/api/java/util/ArrayList.html).

### Internal Structure

![](/images/java-list-1.png) 

In Java, we initialize an ArrayList as 

```java
ArrayList<SampleType> arr = new ArrayList<>();
```

Once we initialized this, it is backed by a normal Java array internally as,

```java
Object[] elementData= ...
```

Since this is backed by a normal Java array, ArrayList uses Contiguous Memory Space for storage inside the heap. By default, it starts with a capacity of 10 on first add and grows by 1.5X when space becomes occupied. 

```text
new_capacity = old_capacity + old_capacity >> 1
```

But if we roughly know the number of elements that will be included in the ArrayList, then we can define an ArrayList with custom capacity, as shown below, which improves efficiency by avoiding requirement for unnecessary growth.

```java
ArrayList<SampleType> arr = new ArrayList<>(25);
```

Java Generic Collections (Including ArrayList) do not support primitive data types. Primitive data is autoboxed when it is being added to Generic Collections. It only stores object references in an ArrayList, which takes,

```text
32-bit System - 4 bytes
64-bit System - 4 bytes with CompressedOops
                8 bytes without CompressedOops
```

Also, it should be noted that ArrayLists are not thread-safe.

### Random Access

Since ArrayList uses contiguous memory space, it's very efficient to get an object at a given index. Because the location of the object reference pointer inside the memory can be calculated based on direct memory indexing as follows,

```text
memeory_address_of_object = memeory_address_of_arr_start + (index x size_of_each_element_pointer)
```

Due to this, it gives **O(1)** time complexity when we try to get an object at an index.

### Insert at End

![](/images/java-list-2.PNG)   

When we try to insert at the end, 

-  Calculate the end memory location of the array
-  Check capacity filling
   -   If need growth  => grows by 1.5X => need element copying to new location with more space => **O(n)**
   -   If no growth needed => Just add at end => **O(1)**

In general array list has **O(1)** for adding an element at the end, worst case **O(n)**, when resizing.

### Remove from End

![](/images/java-list-3.PNG)   

When we try to delete from the end, 

- Calculate the end memory location of the array
- Delete the end element and set the removed slot to null to avoid memory leaks

In general array list has **O(1)** for deleting an element from the end.

### Insert at Start/Middle

![](/images/java-list-4.PNG)   

However, when we try to enter an element at the beginning/middle, we can't enter directly since, ArrayList is backed by contiguous memory space.

So when it comes to inserting in the beginning/middle,

- Should grow the array size (need element copying to a new location with more space) if the capacity is filled.
- Shift all the elements to the right.
- Insert a new element at the beginning/middle.

This operation also has **O(n)** time complexity. So, ArrayList is not a good choice if we have frequent insertions at the beginning/middle.

### Remove from Start/Middle

![](/images/java-list-5.PNG)   

Since this uses contiguous memory spaces, when we try to delete an element from the beginning/middle,

- First, we need to remove that object. 
- Shift all elements to the left.

This operation also has **O(n)** time complexity. So ArrayList is not a good choice if we have frequent deletetion from begining/middle.

In summary, ArrayList

```text
Random Access, Frequent Insert/Delete End -> Optimal ✅
Frequent Insert/Delete Beginning/Middle -> Not Optimal ❌
```
---
## Linked List

Now, we will be focusing on LinkedList, another list type in Java. For more details about the functions it supports, please refer to [documentation](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedList.html).

### Internal Structure

![](/images/java-ll-1.PNG)                                                      

In Java, we initialize a LinkedList as 

```java
LinkedList<SampleType> arr = new LinkedList<>();
```

Once we initialized this, it is backed by a chain of Node Objects, where each node has references for next and previous Nodes in the chain. And each node holds the value of the stored element. We simply call this chain as doubly-linked list.

```java
private static class Node<T>{
  T item;
  Node<T> next;
  Node<T> prev;
}
```

Since this is backed by a set of objects in memory, the structure is scattered across the heap; in other words, the memory is not contiguous. Due to the structure of LinkedList, it can grow dynamically, and no initial capacity provisioning is needed like in ArrayList. This prevents it from having extra growth and resizing costs like in ArrayList.

The reference of the start and end nodes will be stored inside the stack, as shown in the image.

Also, it should be noted that LinkedLists are also not thread-safe.

### Random Access 

![](/images/java-ll-2.PNG)                         

LinkedList uses a chain of node objects scattered across the memory, and initially, we only have the reference to the start and end nodes. Due to this, we have to start from the starting/ending node and traverse through each node to find the node at a given index. This gives a time complexity of **O(n)**.

So LinkedList is not optimal if we randomly access element in a larger collection frequently.

### Insert/Remove at Start/End

![](/images/java-ll-3.PNG)                          
![](/images/java-ll-4.PNG)       
![](/images/java-ll-5.PNG)    
![](/images/java-ll-6.PNG)  

Since we know the node object references of start and end nodes, we just need to create/remove a node at start/end and update the reference pointer. This operation takes a constant time complexity of **O(1)** regardless of the array size.

### Insert/Remove from Middle

![](/images/java-ll-7.PNG)       
![](/images/java-ll-8.PNG)    
![](/images/java-ll-9.PNG)                 

Assume we are inserting/removing an element at a middle index. We have to

- Create a node with an element - O(1)
- Find Node. Since we only know the node references of start and end, first we have to traverse and find the n-th node, which takes - O(n)
- Update pointer accordingly - O(1)

This gives us **O(n)** time complexity for insertion and deletion at the middle.

In summary, LinkedList
```text
Frequent Insert/Delete Start/End -> Optimal ✅
Random Access, Frequent Insert/Delete Middle -> Not Optimal ❌
```
---
## Summary

| Use Case             	| ArrayList                                     	| LinkedList                       	|
|----------------------	|-----------------------------------------------	|----------------------------------	|
| Internal Structure   	| Contiguous Array in Memory                    	| Doubly Linked Node Chain         	|
| Memory Usage         	| Low                                           	| High                             	|
| When to Use          	| Random Access, Frequenct Insert/Delete at End 	| Frequent Insert/Delete Start/End 	|
| Random Access        	| O(1)                                          	| O(n)                             	|
| Insert/Delete Start  	| O(n)                                          	| O(1)                             	|
| Insert/Delete Middle 	| O(n)                                          	| O(n)                             	|
| Insert/Delete End    	| O(1)                                          	| O(1)                             	|

---
## Immutable List

If we want a read-only version of the List Collection, we can use List.of(), List.copyOf(), Collections.unmodifiableList(...).

---
> Please note that collections we described here are **not Thread-Safe**.


Thank You !\
Happy Coding 🙌