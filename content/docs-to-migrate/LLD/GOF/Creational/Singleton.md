---
weight: 100
---
# GOF Design Patterns : Singleton Design Pattern

![](/images/Screenshot%202025-01-29%20092951.png)

## Introduction

In this blog post, we are going to discuss one of the popular design patterns that come under the Creational Design Pattern, the Singleton Design Pattern. The aim of this design pattern is to ensure that only one instance of its kind exists for a given class within the context of the entire application runtime. It also exposes a single point of access to it from anywhere in the application.

So, in summary, this lets you access your object from anywhere in the application while encapsulating the attributes of this application within a single class, Guarantees that only one instance of this class is available at any point in time. The class diagram is as follows.

## Class Diagram

![](/images/Singleton_pattern_uml.png)

## Let's Code

Now let's see how we can achieve this step by step. Let's first begin with basic step.
```java
public class ExampleClass {

    // Note 1
    private static ExampleClass instance;
    private String someAttribute;

    // Note 2
    private ExampleClass(String someAttribute) {
        this.someAttribute = someAttribute;
    }

    // Note 3
    public static ExampleClass getInstance(String someAttribute) {

        // Note 4
        if (instance == null) {
            instance = new ExampleClass(someAttribute);
        }
        // Note 5
        return instance
    }
}
```
- Note 1: First, we create a field of the class itself and make it static. So no matter how many objects get created, all the objects refer to the same instance. Then, we also make that field private to prevent direct access to that instance location.
- Note 2: As the next step, we make the constructor private, ensuring that no one else can create an object instance from outside the class.
- Note 3: Then, we introduce a static method getInstance\(\). Those who need an instance of this object should call this method.
So far, we have created some access control around instance initialization and its access logic.
Now, let's focus on instance generation.
- Note 4: Before we create an instance, we first check whether an instance has already been created for the given class. If not, we create a new instance and store it in our instance field.
- Note 5: Eventually, we return the instance reference to the requester of the instance.

As we can see, the pattern seems straightforward. However, it has some issues we should address. If we consider a single\-threaded environment, the logic will work fine.

However, if we consider a multithreaded environment, what if two threads access the getInstance\(\) method at the same time and perform the null check simultaneously? Both threads might think the object is not initialized yet and create new instances. So how do we prevent this issue in a multithreaded environment?

## Muti Threads

Let's try to wrap our if method with a synchronized block. Now, only one thread will execute the null check and initialize the object at a given time. Now, we have solved the threading issue.

Here is how updated code looks like.

```java
public class ExampleClass {

	private static ExampleClass instance; 
	private String someAttribute;
	
	private ExampleClass(String someAttribute) {
		this.someAttribute = someAttribute;
	}
	
	public static ExampleClass getInstance(String someAttribute) {
		
		// Note 6
		synchronized (ExampleClass.class) {
		
			if(instance==null) {
				instance = new ExampleClass(someAttribute);
			}
			
		}
		
		return instance; 
	}
}
```

- Note 6 : Now we added this sync block so only on thread is allowed to check null and initialize an object at a give time.

However, if we think properly, this is an unnecessary overhead on performance since every thread has to go through this synchronized block one after another, regardless of whether the object is already initialized or not.

## Improve Performance

Let's try to improve this a bit. We can use the double\-checked locking idiom in our method to solve the issue.

So, we can introduce a new condition where we only need to worry about multithreading when there is no instance created. Once an instance is created in a thread\-safe manner, we no longer need to worry about synchronization or initialization logic.

The solution is simple: we just need to wrap the synchronized block with another null check.

```java
public class ExampleClass {

    private static ExampleClass instance;
    private String someAttribute;

    private ExampleClass(String someAttribute) {
        this.someAttribute = someAttribute;
    }

    public static ExampleClass getInstance(String someAttribute) {

        // Note 7
        if (instance == null) {
            synchronized(ExampleClass.class) {

                if (instance == null) {
                    instance = new ExampleClass(someAttribute);
                }

            }
        }

        return instance;
    }
}
```
- Note 7 : Simply, we improve the performance of our code by limiting synchronization to only the rare case where two threads compute null at the same time and attempt to construct the object simultaneously. In common cases, where the instance is already created, we skip synchronization, making the code more efficient.

## Caching is an Issue ?

#### So, everything is fine? 
Actually, no. It still misses a fix for a rare case.

Considering thread\-level caching, what if Thread 1 initializes the object, but before it reflects in the main memory, Thread 2 reads from its cache? Thread 2 checks for null, thinks no one has initialized the instance yet, and starts the initialization process. This is a rare case where issues can occur.

The solution is simple: we need to prevent threads from caching the instance variable. Instead, all reads and writes should directly happen in the main memory.

#### How do we do this?
We can use the volatile keyword in Java. Declaring the instance variable as volatile ensures that:

- Every read and write happens directly in main memory.
- No thread reads a stale cached value.

```java
public class ExampleClass {
    // Note 8
    private static volatile ExampleClass instance;
    private String someAttribute;

    private ExampleClass(String someAttribute) {
        this.someAttribute = someAttribute;
    }

    public static ExampleClass getInstance(String someAttribute) {

        if (instance == null) {
            synchronized(ExampleClass.class) {

                if (instance == null) {
                    instance = new ExampleClass(someAttribute);
                }

            }
        }

        return instance;
    }
}
```

- Note 8 : Volatile ensures visibility: Changes made to instance by one thread are immediately visible to other threads. Also It prevents instruction reordering, ensures object initialization happens before assigning the reference to instance.

## Again Improve Performance

Now, we have a robust solution for the Singleton object. However, we got rid of caching, which is a performance issue.

To solve this, we can ask the thread to use a local variable to refer to the instance variable in main memory. By using this local variable, the thread can cache it, improving performance while maintaining thread safety.
```java
public class ExampleClass {

    private static volatile ExampleClass instance;
    private String someAttribute;

    private ExampleClass(String someAttribute) {
        this.someAttribute = someAttribute;
    }

    public static ExampleClass getInstance(String someAttribute) {

        ExampleClass localInstance = instance; // Note 9

        if (localInstance == null) { // Note 9
            synchronized(ExampleClass.class) {

                localInstance = instance; // Note 9

                if (localInstance == null) { // Note 9
                    instance = localInstance = new ExampleClass(someAttribute); // Note 9
                }

            }

        }

        return localInstance; // Note 9
    }
}
```
- Note 9 : Now, we have created a localInstance at line 12 to store a reference to our volatile instance field. This allows threads to cache localInstance while maintaining thread safety. At line 14, we check for null using that local reference. If it's null, we proceed with the synchronized block. After blocking other threads from accessing the synchronized block, we fetch the instance again at line 17 and check for null at line 19 to ensure that no other thread has created an instance in the meantime. Then, we update both the local variable and static volatile variable with our new object reference at line 20. Finally, at line 27, we return the instance to the requester.

Now we got a robust solution for the Singleton Design Pattern with both thread safety and optimized performance\! 🚀

## When to Use Singleton
- Database Connection Manager : Ensures a single instance of the database connection pool is shared across the application to optimize resource usage.
- Logger Utility : Provides a single logging instance to ensure consistent logging behavior across the application.
- Configuration Manager : Centralizes access to application\-wide settings or configurations, avoiding redundant reloading of settings.
- Thread Pool Manager : Manages a shared pool of threads to optimize thread creation and reuse, ensuring efficient multithreading.
- Caching System : Maintains a single instance of a cache \(e.g., in\-memory cache\) to store frequently accessed data for improved performance.

Yup. That's it for Now 

Happy Coding 🙌
