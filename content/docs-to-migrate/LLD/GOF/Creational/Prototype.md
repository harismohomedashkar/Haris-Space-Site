---
weight: 500
---
# GOF Design Patterns : Prototype

![](/images/Screenshot%202025-02-20%20165133.png)

## Introduction

In our application there can be Objects, which takes lot of resources and effort to initialize and build up to some extent. These types of objects creation are costly that requires a lot of time and resources. But if we already created a such Object, if we want more Objects like, what we do ? create from scratch again or using same Object to just clone it. It is the design decision, which involves Prototype Design Pattern. Prototype Design Pattern is mechanism to copy the original object to a new object and modify it according to our needs.

## Example

Let's start with an example.
Let's say we get a request from user, where user uploads a image file, and our system is suppose to send PNG and JPEG version of that image file. Let's build flow for two approaches we have,

### First Option
- Get file from user
- Create 2 objects \(with same user file content\) for each file type that we suppose to send \(consumes more memory\)
- Send each object to file scanner and scan for vulnerabilities \(costly task and invoked 2 times\)
- Convert each scanned objects to preferred target file type


![](/images/image1.png)
**OR**

### Second Option
- Get file from user
- Put file content into 1 Object \(now less memory used\)
- Send that object to scan and  Get results \(costly task but invoked only 1 time\)
- Now clone into 2 copies for we needed \(PNG & JPEG\)
- Convert and send results

![](/images/image1.png)

### Better One

As we can see second approach is more sound and efficient. However it need some mechanism to clone object. This is where we can use protype design pattern.

As we concluded The better approach would be to clone the existing object into a new object and then do the data manipulation to reduce number of times costly tasks invocation and save more memory.

## Shallow vs Deep

When is come to prototyping there is two ways we can do
1. **Shallow Copy** : Creates a new instance of the object but shares references of nested objects \(if any\), Changes in nested objects affect both the original and the cloned object. Primitive fields like int, double, boolean, etc., are not shared in a shallow copy.

2. **Deep copy** : Creates a new instance of the object and its nested objects. Make sure that changes in the original object, including nested objects, do not affect the cloned object and its nested objects.


![](/images/image1.png)

## Coding

### Shallow Copy
```java
package Example;

import java.io.File;
import java.time.LocalDateTime;
import java.util.HashMap;

public class PrototypeDesignPattern {

    public static void main(String[] args) throws CloneNotSupportedException {

        // DEFINE ORIGINAL IMAGE AND PRINT
        ImageFile ImageOriginal = new ImageFile(new File("C:\\Users\\HarisMohomedAshkar\\Downloads\\Test.png"));
        ScanResult scanResult = new ScanResult(true, LocalDateTime.now());
        ImageOriginal.addMetaData("Tag", "TestImage_1");
        ImageOriginal.setScanResult(scanResult);
        System.out.println(ImageOriginal.toString());
        // ImageFile{
        // 		size=35445, filePath=C:\Users\HarisMohomedAshkar\Downloads\Test.png, 
        //      metaData={path=C:\Users\HarisMohomedAshkar\Downloads\Test.png, name=Test.png, Tag=TestImage_1}, 
        //      scanResult=ScanResult{isSafe=true, scannedAt=2025-02-14T11:10:29.946809300}
        // }

        // CLONE AND PRINT
        ImageFile ImageClone = ImageOriginal.clone();
        System.out.println(ImageClone.toString());
        // ImageFile{
        // 		size=35445, filePath=C:\Users\HarisMohomedAshkar\Downloads\Test.png, 
        //      metaData={path=C:\Users\HarisMohomedAshkar\Downloads\Test.png, name=Test.png, Tag=TestImage_1}, 
        //      scanResult=ScanResult{isSafe=true, scannedAt=2025-02-14T11:10:29.946809300}
        // }

        // UPDATE SOME NESTED OBJECTS IN ORIGINAL
        scanResult.setSafe(false);
        scanResult.setScannedAt(LocalDateTime.now());

        // ORIGINAL AFTER UPDATED
        System.out.println(ImageOriginal.toString());
        // ImageFile{
        // 		size=35445, filePath=C:\Users\HarisMohomedAshkar\Downloads\Test.png, 
        //      metaData={path=C:\Users\HarisMohomedAshkar\Downloads\Test.png, name=Test.png, Tag=TestImage_1}, 
        //      scanResult=ScanResult{isSafe=false, scannedAt=2025-02-14T11:14:07.838987400}
        // }

        // CLONE AFTER UPDATED
        System.out.println(ImageClone.toString());
        // ImageFile{
        // 		size=35445, filePath=C:\Users\HarisMohomedAshkar\Downloads\Test.png, 
        //      metaData={path=C:\Users\HarisMohomedAshkar\Downloads\Test.png, name=Test.png, Tag=TestImage_1}, 
        //      scanResult=ScanResult{isSafe=false, scannedAt=2025-02-14T11:14:07.838987400}
        // }

        // CHANGES IS NESTED OBJECTS IN ORIGINAL IMAGEFILE, AFFECTS CLONED IMAGEFILE

    }

}


class ImageFile implements Cloneable {

    private long size;
    private File file;
    private HashMap < String, String > metaData;
    private ScanResult scanResult;

    public ImageFile(File file) {
        this.size = file.length();
        this.file = file;
        this.metaData = new HashMap < String, String > ();
        this.metaData.put("name", file.getName());
        this.metaData.put("path", file.getPath());
    }

    public long getSize() {
        return size;
    }

    public File getFile() {
        return file;
    }

    public HashMap < String, String > getMetaData() {
        return metaData;
    }

    public ScanResult getScanResult() {
        return scanResult;
    }

    public ImageFile addMetaData(String key, String val) {
        this.metaData.putIfAbsent(key, val);
        return this;
    }

    public ImageFile setMetaData(HashMap < String, String > metaData) {
        this.metaData = metaData;
        return this;
    }

    public ImageFile setScanResult(ScanResult scanResult) {
        this.scanResult = scanResult;
        return this;
    }

    @Override
    public String toString() {
        return "ImageFile{" +
            "size=" + size +
            ", filePath=" + file.getAbsolutePath() +
            ", metaData=" + metaData +
            ", scanResult=" + scanResult +
            '}';
    }

    @Override
    protected ImageFile clone() throws CloneNotSupportedException {
        // Use same File, ScanResult and MetaData References
        ImageFile imageFile = new ImageFile(file);
        imageFile
            .setScanResult(scanResult)
            .setMetaData(metaData);
        return imageFile;
    }
}

class ScanResult {
    private boolean isSafe;
    private LocalDateTime scannedAt;

    public ScanResult(boolean isSafe, LocalDateTime scannedAt) {
        this.isSafe = isSafe;
        this.scannedAt = scannedAt;
    }

    public boolean isSafe() {
        return isSafe;
    }

    public LocalDateTime getScannedAt() {
        return scannedAt;
    }

    public void setSafe(boolean isSafe) {
        this.isSafe = isSafe;
    }

    public void setScannedAt(LocalDateTime scannedAt) {
        this.scannedAt = scannedAt;
    }

    @Override
    public String toString() {
        return "ScanResult{" +
            "isSafe=" + isSafe +
            ", scannedAt=" + scannedAt +
            '}';
    }
}
```
### Deep Copy
```java
    @Override
    protected ImageFile clone() throws CloneNotSupportedException {
        // Create a new Objects For File, ScanResult and MetaData
        File newFile = new File(file.getAbsolutePath());
        ScanResult newScanResult = new ScanResult(scanResult.isSafe(), LocalDateTime.from(scanResult.getScannedAt()));
        HashMap < String, String > newMetaData = new HashMap < > (metaData);
        ImageFile clonedImageFile = new ImageFile(newFile);
        clonedImageFile.setScanResult(newScanResult)
            .setMetaData(newMetaData);
        return clonedImageFile;
    }
```
```java
public class PrototypeDesignPattern {

    public static void main(String[] args) throws CloneNotSupportedException {

        // DEFINE ORIGINAL IMAGE AND PRINT
        ImageFile ImageOriginal = new ImageFile(new File("C:\\Users\\HarisMohomedAshkar\\Downloads\\Test.png"));
        ScanResult scanResult = new ScanResult(true, LocalDateTime.now());
        ImageOriginal.addMetaData("Tag", "TestImage_1");
        ImageOriginal.setScanResult(scanResult);
        System.out.println(ImageOriginal.toString());
        // ImageFile{
        // 		size=35445, filePath=C:\Users\HarisMohomedAshkar\Downloads\Test.png, 
        //      metaData={path=C:\Users\HarisMohomedAshkar\Downloads\Test.png, name=Test.png, Tag=TestImage_1}, 
        //      scanResult=ScanResult{isSafe=true, scannedAt=2025-02-14T11:27:56.571472200}
        // }

        // CLONE AND PRINT
        ImageFile ImageClone = ImageOriginal.clone();
        System.out.println(ImageClone.toString());
        // ImageFile{
        // 		size=35445, filePath=C:\Users\HarisMohomedAshkar\Downloads\Test.png, 
        //      metaData={name=Test.png, path=C:\Users\HarisMohomedAshkar\Downloads\Test.png, Tag=TestImage_1}, 
        //      scanResult=ScanResult{isSafe=true, scannedAt=2025-02-14T11:27:56.571472200}
        // }

        // UPDATE SOME NESTED OBJECTS IN ORIGINAL
        scanResult.setSafe(false);
        scanResult.setScannedAt(LocalDateTime.now());

        // ORIGINAL AFTER UPDATED
        System.out.println(ImageOriginal.toString());
        // ImageFile{
        // 		size=35445, filePath=C:\Users\HarisMohomedAshkar\Downloads\Test.png, 
        //      metaData={path=C:\Users\HarisMohomedAshkar\Downloads\Test.png, name=Test.png, Tag=TestImage_1}, 
        //      scanResult=ScanResult{isSafe=false, scannedAt=2025-02-14T11:27:56.590477700}
        // }

        // CLONE AFTER UPDATED
        System.out.println(ImageClone.toString());
        // ImageFile{
        // 		size=35445, filePath=C:\Users\HarisMohomedAshkar\Downloads\Test.png, 
        //      metaData={path=C:\Users\HarisMohomedAshkar\Downloads\Test.png, name=Test.png, Tag=TestImage_1}, 
        //      scanResult=ScanResult{isSafe=true, scannedAt=2025-02-14T11:27:56.571472200}
        // }

        // CHANGES IS NESTED OBJECTS IN ORIGINAL IMAGEFILE & CLONED IMAGEFILE ARE INDEPENDENT

    }

}
```

Here we have looked how we can use prototype design pattern and clone a shallow or deep copy for an existing objects to make our application more efficient.

Happy Coding 🙌
