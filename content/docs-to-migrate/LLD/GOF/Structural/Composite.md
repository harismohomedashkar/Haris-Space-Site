---
# 786
weight: 400
---
# GOF Design Patterns : Composite

## Introduction

According to GOF, the Composite Design Pattern falls under Structural Design Patterns because this pattern focuses on how we compose hierarchical object structures to process data efficiently.

According to GOF definition,
```text
Compose objects into tree structure to represent part-whole hierarchies. Composite lets client treat individual objects and compositions of objects uniformly  
```

## When to Use

This pattern is used in cases where,
- There is hierarchical object storage, where an object holds other objects in the next level of the hierarchy like a tree structure.
- The client program uses only common methods across all objects in the hierarchy and processes each node similarly.

## Example

One of the best examples of using this pattern is how we process File Storage in a system.

File storage usually has two types of objects,

- `File` : A single unit of the hierarchy with no children (we call this a leaf)
- `Folder` : A composite object that can hold other child objects (single or composite) in the hierarchy (we call this a composite)

As we know, there are many methods common to files and folders. For our program, we use `display()`, `getSize()`, and `displayList()` methods, which are common to both files and folders. Functions are as follows`,

- `displayList()` : Returns the structure and size of elements in the underlying tree as a string array
- `display()` : Prints the structure and size of elements in the underlying tree
- `getSize()` : Returns the total size of all underlying elements in the tree

### Component
First, we need to define an interface to store these common methods. This interface is called the `Component`.
```java
sealed interface StorageEntity permits File, Folder {

	List<String> displayList();

	double getSize();

	void display();

}
```
### Leaf
Next, we define our leaf object, which is the simplest unit and does not hold children. In our example, this is `File`.
```java
final class File implements StorageEntity {

	String name;
	double size;

	public File(String name, double size) {
		this.name = name;
		this.size = size;
	}

	@Override
	public List<String> displayList() {
		return List.of("- " + this.name + " : " + new DecimalFormat("##.00").format(getSize()) + " KB");
	}

	@Override
	public double getSize() {
		return this.size;
	}

	@Override
	public void display() {
		System.out.println(displayList().get(0));
	}

}
```
### Composite
Next, we define our composite object, which processes itself and all its children
```java
final class Folder implements StorageEntity {

	String name;
	private List<StorageEntity> entities;

	public Folder(String name) {
		this.name = name;
		this.entities = new ArrayList<StorageEntity>();
	}

	@Override
	public List<String> displayList() {
		List<String> lines = new ArrayList<String>();
		lines.add("[D] " + this.name + " [T] " + new DecimalFormat("##.00").format(getSize()) + " KB");
		for (StorageEntity storageEntity : entities) {
			for (String line : storageEntity.displayList()) {
				lines.add("    " + line);
			}
		}
		return lines;
	}

	@Override
	public double getSize() {
		double sumSize = 0;
		for (StorageEntity storageEntity : entities) {
			sumSize = sumSize + storageEntity.getSize();
		}
		return sumSize;
	}

	@Override
	public void display() {
		for (String line : displayList()) {
			System.out.println(line);
		}
	}

	public void addEntity(StorageEntity entity) {
		this.entities.add(entity);
	}

}
```
### Client
Now let's build a simple storage hierarchy to test our `Composite Design Pattern` example.
```java
class Client {

	public static void main(String[] args) {

		File beach = new File("beach.png", 11.2);
		File dolphin = new File("dolphin.png", 12.4);
		File oak = new File("oak.png", 10.1);
		File mango = new File("mango.png", 9.6);
		File eagle = new File("eagle.jpeg", 1.1);

		File ss1 = new File("ss1.jpeg", 1.1);
		File ss2 = new File("ss2.jpeg", 2.3);
		File ss3 = new File("ss3.jpeg", 1.7);

		Folder root = new Folder("ROOT");
		Folder cameraRoll = new Folder("CAMERA ROLL");
		Folder screenShot = new Folder("SCREENSHOTS");
		Folder tree = new Folder("TREE");
		Folder bird = new Folder("BIRD");
		Folder fish = new Folder("FISH");

		root.addEntity(cameraRoll);
		root.addEntity(screenShot);

		cameraRoll.addEntity(fish);
		cameraRoll.addEntity(tree);
		cameraRoll.addEntity(bird);
		cameraRoll.addEntity(beach);

		screenShot.addEntity(ss1);
		screenShot.addEntity(ss2);
		screenShot.addEntity(ss3);

		fish.addEntity(dolphin);
		tree.addEntity(oak);
		tree.addEntity(mango);
		bird.addEntity(eagle);

		root.display();

	}

}
```
### Output
Here is what we get as our output,
```text
[D] ROOT [T] 49.50 KB
    [D] CAMERA ROLL [T] 44.40 KB
        [D] FISH [T] 12.40 KB
            - dolphin.png : 12.40 KB
        [D] TREE [T] 19.70 KB
            - oak.png : 10.10 KB
            - mango.png : 9.60 KB
        [D] BIRD [T] 1.10 KB
            - eagle.jpeg : 1.10 KB
        - beach.png : 11.20 KB
    [D] SCREENSHOTS [T] 5.10 KB
        - ss1.jpeg : 1.10 KB
        - ss2.jpeg : 2.30 KB
        - ss3.jpeg : 1.70 KB
```
## Wrap-up
As discussed so far, this pattern is about handling tree like hierarchical structures to,
- Create complex structures dynamically and process them efficiently.
- Reduce duplicate code by allowing processing of each element through a common Component interface.
- Create a hierarchy once and process it in multiple ways as needed, saving memory and time by avoiding recreation of new objects each time.

Hope this article was helpful.\
Thank you !\
Happy Coding 🙌