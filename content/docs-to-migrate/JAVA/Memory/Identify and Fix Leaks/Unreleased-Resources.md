---
weight: 300
---

# Java Memory Leaks : Unreleased Resources

## Problem

Let's assume we have resource files and want to process them, and based on the results, we will proceed to the next step. However, let's assume we somehow didn't close those resources before proceeding to the next step, as shown below.

```java
public class ResourceCloseSimulation {
    public static void main(String[] args) throws InterruptedException, IOException {
        Thread.sleep(10000);
        System.out.println("Loading Resources . . .");
        int count = 0;
        while (true) {
            String path = BlacklistIpGenerator.getIpFile("output", count);
            FileInputStream fis = new FileInputStream(path);
            IPBlackListFileHandler.processStream(fis);
            count++;
            if (count % 100 == 0) {
                System.out.println("Opened " + count + " files");
            }
        }
    }
}
```

And keep opening such resource files without closing. Each time we open a resource file, the OS allocates a `file descriptor` for that file and releases those descriptors only when the file is closed. However, OS has a limit of `file descriptor` which is allocated for a given process. If we keep opening the files without closing, eventually the limit will be exceeded, and we will get an error like,

> **java.io.FileNotFoundException: output_999.txt (Too many open files)**

To prevent such issues. It is best practice to close such resource streams after processing. We can do these in two ways,

## Fix 1

Use `.close()` to release the resource

```java
public class ResourceCloseSimulation {
    public static void main(String[] args) throws InterruptedException, IOException {
        Thread.sleep(10000);
        System.out.println("Loading Resources . . .");
        int count = 0;
        while (true) {
            String path = BlacklistIpGenerator.getIpFile("output", count);
            FileInputStream fis = new FileInputStream(path);
            IPBlackListFileHandler.processStream(fis);
            fis.close()
            count++;
            if (count % 100 == 0) {
                System.out.println("Opened " + count + " files");
            }
        }
    }
}
```

## Fix 2 (Recommended)

Wrap the Resource Handler with the `AutoClosable` interface and use try processing within a `try block`.

```java
public class ResourceCloseSimulation {
    public static void main(String[] args) throws Exception {
        Thread.sleep(10000);
        System.out.println("Loading Resources . . .");
        int count = 0;
        while (true) {
            String path = BlacklistIpGenerator.getIpFile("output", count);
            try (IPBlackListFileStream ipBlackListFileStream = new IPBlackListFileStream(path)) {
                ipBlackListFileStream.processBytes();
            }
            count++;
            if (count % 100 == 0) {
                System.out.println("Opened " + count + " files");
            }
        }
    }
}

class IPBlackListFileStream implements AutoCloseable {
    FileInputStream fis;
    public IPBlackListFileStream(String path) throws FileNotFoundException {
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

Each way, the file descriptor will be released once it's being processed, which prevents the system from memory leak.

Happy Coding 🙌