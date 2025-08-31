---
# 786
weight: 500
---
# GOF Design Patterns : Proxy

## Introduction

Proxy Design Pattern is one of the GOF Design Patterns, which falls under the Structural Design Patterns. We can define the Proxy Design Pattern as,

> A pattern where we create a placeholder object for the original object to control access to the original object.

The main purpose of this pattern is to make the client interact with the original object through an intermediary object (proxy object),
which updates the invocation to the original object and response from the original object, without updating the original object's logic.

## Structure

In the Proxy Pattern, there are 3 Actors, we can see,

1. `Subject Interface` --> Interface with defined common methods that need to be proxied.
2. `Real Subject` --> Original class that implements methods with real logic.
3. `Proxy Subject` --> Proxy class, which holds a reference to the Original class, and implements methods with modified logics.

## Example

Let's use a Lazy Load Image example to get a better understanding of this Proxy Design Pattern. Assume we have the following classes defined.

![](/images/proxy_uml.png)

Subject Interface
```java
interface Image {
	void showImage();
	String getFileName();
}
```
Real Subject
```java
class RealImage implements Image {

	private String fileName;
	private Object imageData;

	public RealImage(String fileName) {
		this.fileName = fileName;
		this.imageData = loadFromS3();
	}

	@Override
	public void showImage() {
		System.out.println(String.format("Image %s loaded from S3 with HashCode of Object %s", this.fileName,
				this.imageData.hashCode()));
	}

	@Override
	public String getFileName() {
		return this.fileName;
	}

	private Object loadFromS3() {
		return new Object(); // Just For Demo
	}

}
```
Proxy Subject
```java
class ProxyImage implements Image {

	private String fileName;
	private RealImage realImage;

	public ProxyImage(String fileName) {
		this.fileName = fileName;
	}

	@Override
	public void showImage() {
		if (realImage == null)
			realImage = new RealImage(this.fileName);
		realImage.showImage();
	}

	@Override
	public String getFileName() {
		return this.fileName;
	}

}
```

As shown in the code below, users can load images not to view, but just see the image names before selection. In such a case, we can use a Virtual Proxy to
avoid fetching real image data for all the images the user requests. And get details from our storage only when the user really requests the image to view.

Client
```java
public class ProxyDemo {

	public static void main(String[] args) {
		Image img1 = new ProxyImage("IMG-1.png");
		Image img2 = new ProxyImage("IMG-2.png");
		Image img3 = new ProxyImage("IMG-3.png");
		Image img4 = new ProxyImage("IMG-4.png");
		Image img5 = new ProxyImage("IMG-5.png");
		Image img6 = new ProxyImage("IMG-6.png");
		Image img7 = new ProxyImage("IMG-7.png");

		List<Image> images = List.of(img1, img2, img3, img4, img5, img6, img7);

		// User Prints Name of All 7 Images
		for (Image img : images) {
			System.out.println(img.getFileName());
		}

		// But User Decided to Click Only 5th Image
		images.get(4).showImage();
	}

}
```
Output
```text
IMG-1.png
IMG-2.png
IMG-3.png
IMG-4.png
IMG-5.png
IMG-6.png
IMG-7.png
Image IMG-5.png loaded from S3 with HashCode of Object 1867083167
```

## Usage

Likewise, these intermediary logics (in proxy object), which wrap around the original call, can be used to

### Access Control
We can define a Protection Proxy to add layers of security, such as IP filtering, rate limiting, etc., to make sure only safe calls get a chance to invoke the real object logic.

### Resource Management
There can be some objects that are costly to initialize (maybe due to the need for DB calls, external API calls, large sizes, complex calculations, etc.). We can introduce a Virtual Proxy to allow the client to use this proxy object in place of the costly object, and let the real initialization of this costly object be invoked only when the data is actually needed. This is called lazy loading.

### Remote Access 
If we have a remote object residing on a different server, we can use a proxy object to make this object look local to the client. The client can use the proxy object as a local representation, while the proxy handles the calls to the remote server internally.

### Logging and Monitoring
We can define a proxy object to log invocation details before invoking the real call, and response details before returning the original object’s response (such as time taken to respond, etc.) back to the client. Using this, we can perform auditing, collect performance metrics, and do analysis.

### Caching
Assume we are frequently calling some object method with the same parameters, and the calculation of the result is costly. In such cases, we can maintain the input parameters and output results in a separate cache by introducing a Cache Proxy. This boosts the performance of the application by reducing repeated calculation time.

That's all for Proxy Pattern for now.

Hope this article was helpful.\
Thank you !\
Happy Coding 🙌