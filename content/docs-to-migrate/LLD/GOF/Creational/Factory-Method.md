---
weight: 200
---
# GOF Design Patterns : Factory Method

![](/images/Screenshot%202025-02-04%20080056.png)

## Problem

Let's start with an example. Suppose we want to create an ImageReader class. This class is supposed to read multiple types of images, and support for image types can be added or removed later as the project expands. For example, let's say it currently supports PNG and JPEG formats. Refer to the following code:

```java
abstract class Image {
	public abstract void readBytes(String path);
}

class PNGImage extends Image {
	@Override
	public void readBytes(String path) {
		System.out.println(String.format("Reaing PNG Image at %s", path));
	}
}

class JPEGImage extends Image {
	@Override
	public void readBytes(String path) {
		System.out.println(String.format("Reaing JPEG Image at %s", path));
	}
}

class ImageReader {

	private Image imageType;

	public ImageReader(String type) {
		switch (type) {
		case "PNG": {
			imageType = new PNGImage();
			break;
		}
		case "JPEG": {
			imageType = new JPEGImage();
			break;
		}
		default:
			throw new IllegalArgumentException("Unsupported value: " + type);
		}
	}

	public void readImage(String path) {
		imageType.readBytes(path);
	}
}

public class Example {
	public static void main(String[] args) {
		ImageReader reader1 = new ImageReader("PNG");
		reader1.readImage("C:\\Some\\Sample\\Path1");

		ImageReader reader2 = new ImageReader("JPEG");
		reader2.readImage("C:\\Some\\Sample\\Path2");
	}
}
```
Now, let's assume we need to add support for a new image type, BMP. This can be done simply by adding a new class for the BMP image, which extends the abstract class Image, and adding a new case statement to initialize it. The updated code looks as follows:
```java
// Adding New BMP Image Class
class BMPImage extends Image {
	@Override
	public void readBytes(String path) {
		System.out.println(String.format("Reaing BMP Image at %s", path));
	}
}

class ImageReader {

	private Image imageType;

	public ImageReader(String type) {
		switch (type) {
		case "PNG": {
			imageType = new PNGImage();
			break;
		}
		case "JPEG": {
			imageType = new JPEGImage();
			break;
		}
		case "BMP": {
			imageType = new BMPImage(); // Adding New CASE Statement
			break;
		}
		default:
			throw new IllegalArgumentException("Unsupported value: " + type);
		}
	}

	public void readImage(String path) {
		imageType.readBytes(path);
	}
}
```
However, there is an issue: we have modified the ImageReader logic. This violates the SOLID principle of "Open for Extension, Closed for Modification." So how can we solve this issue while complying with SOLID principles?

## Fix It

Here comes the usage of the Factory Method design pattern. We can update the code as follows:

1. Define a base interface with all the methods that an ImageReader for any image type should support.
2. Create different ImageReader implementations for different image types with relevant logic.
3. Finally, use the relevant factory to read our image.

```java
abstract class Image {
	public abstract void readBytes(String path);
}

class PNGImage extends Image {
	@Override
	public void readBytes(String path) {
		System.out.println(String.format("Reaing PNG Image at %s", path));
	}
}

class JPEGImage extends Image {
	@Override
	public void readBytes(String path) {
		System.out.println(String.format("Reaing JPEG Image at %s", path));
	}
}

interface ImageReader {
	public void readImage(String path);
}

class PNGImageReader implements ImageReader{
	@Override
	public void readImage(String path) {
		Image pNGImage = new PNGImage();
		pNGImage.readBytes(path);	
	}
}

class JPEGImageReader implements ImageReader{
	@Override
	public void readImage(String path) {
		Image jpegImage = new JPEGImage();
		jpegImage.readBytes(path);	
	}
}

public class Example {
	public static void main(String[] args) {
		ImageReader reader1 = new PNGImageReader();
		reader1.readImage("C:\\Some\\Sample\\Path1");

		ImageReader reader2 = new JPEGImageReader();
		reader2.readImage("C:\\Some\\Sample\\Path2");
	}
}
```
Now, let's add our new requirement: support for BMP images. This time, we do not need to modify any existing classes. We just extend the application with new classes without touching the existing code. The updated code looks as follows:
```java
// Added Image Type
class BMPImage extends Image {
	@Override
	public void readBytes(String path) {
		System.out.println(String.format("Reaing BMP Image at %s", path));
	}
}

// Added Image Reader Class
class BMPImageReader implements ImageReader{
	@Override
	public void readImage(String path) {
		Image bmImage = new BMPImage();
		bmImage.readBytes(path);	
	}
}
```

## Use Cases

Here are the key use cases of the Factory Method design pattern:
- It is utilized in JDBC for establishing connections and in frameworks like Spring for managing beans.
- UI libraries such as Swing and JavaFX leverage factories to create adaptable UI components.
- Logging tools like Log4j depend on factories to generate configurable loggers.
- Factories are useful for creating objects from serialized data, enabling support for multiple formats.

## Class Diagram

![](/images/Untitled%20Diagram.drawio%20(1).png)

Happy Coding 🙌
