---
weight: 700
---

# Java Memory Leaks : Overridden Finalizers

## Problem

Assume we have a file handler class to manage common functions we do on files. And assume that the class code looks as below,

```java
public class FileHandler {

    FileInputStream fis;

    public FileHandler(String path) throws FileNotFoundException {
        fis = new FileInputStream(path);
    }

    public void processBytes() throws IOException {
        byte[] bytes = fis.readAllBytes();
        // Some Process Logic
    }

    @Override
    protected void finalize() throws Throwable {
        fis.close();
        super.finalize();
    }
}
```

As shown above, for proper cleanup, we overrode the `finalize()` method and closed the stream before removing the object. But it is not recommended to override the finalize method, because it can lead to a memory leak.

When an Object has overridden `finalize()` method **Garbage Collector does not collect it immediately**. Instead, it will be sent to **the finalization queue**, which is handled by only **single finalization thread** and garbage collected only after finalization. Assume we sent a lot of objects that have overridden finalizations to a queue, which is handled by **a single thread**. This can easily lead to a bottleneck and cause a delay in garbage collection, which can lead to a spike in memory and cause a memory leak.

## Fix

As a solution to this Finalization is deprecated in Java 9 and removed in Java 18, and `AutoClosable` interface is introduced in Java 7.

Now let's rewrite the above example with `try-with-resources` and `AutoClosable`

First, we need to implement `AutoClosable`

```java
public class FileHandler implements AutoCloseable {

    FileInputStream fis;

    public FileHandler(String path) throws FileNotFoundException {
        fis = new FileInputStream(path);
    }

    public void processBytes() throws IOException {
        byte[] bytes = fis.readAllBytes();
        // Some Process Logic
    }

    @Override
    public void close() throws Exception {
        fis.close();
    }

}
```

And in the main class, we can limit the lifetime of the resource using try-with-resources
```java
	public static void main(String[] args) throws Exception {
	    try (FileHandler fileHandler = new FileHandler("blacklistedips.txt")) {
	        fileHandler.processBytes();
	    }
	}	
```

This recommended approach makes memory usage more efficient and prevents the system from memory leaking, because unlike `finalize()`, there is no queue when using `close()`, and the resource gets closed as soon as the block ends.

Happy Coding 🙌