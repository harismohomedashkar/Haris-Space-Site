---
weight: 100
---
# How to implement plugin support for a Java Application : Part 1

![](/images/0_Lw-e0GtUP8UtBCoq.webp)

## Introduction

With the development of technology, today we can see vast amounts of software applications with different capabilities. And most of these products have a huge customer base. However, the requirements of these customers are changing with the time. Due to this, the developers can’t include almost all the features at the initial stage of the product. Moreover, releasing a new version after a new feature is not a good solution. There can be users who only need a few basic features. Most of the system resources of these types of users will be consumed by these unused features.

As a solution to these problems we can introduce a plugin based approach. Where, we can deliver the application with basic features. And when a user needs an additional feature it can be added to the product as a plugin. Before digging into deep, let’s see what are the benefits we can achieve via plugin based architecture. Here I have listed some of the prominent advantages of this approach.

* Application can be extended dynamically
* Developers can work on different features parallelly
* Plugins are easy to maintain
* Each plugin focuses on specific area
* Plugins can be easily replaceable

As the next step, I am going to implement a plugin based java application. Note that, there are various approaches that can be used to develop plugin based java applications. However in this article, I am using a simple approach based on “java class loader”.

## Implementation

This is a java application which can be used to draw a circle, given the radius as an input. The initial application only supports a circle type called “basic”. However, users can add more circle types as plugins.

Let’s code 💻

### Initializing Project

As the first step I created a maven parent project as shown below.

![](/images/1_7_uSURRxXMsk9c-DlkbUgg.webp)

Here is how the pom.xml is configured.
```xml
<project
	xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
        https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.drawer.circle</groupId>
	<artifactId>circle_drawer</artifactId>
	<version>0.0.1</version>
	<packaging>pom</packaging>
	<modules>
		<module>app</module>
	</modules>
</project>
```

Since this is a parent project, the packaging is configured as pom. I have added a maven module app to this parent project. Here are the purposes of those files and folders.
1. pom.xml : Configure parent project and sub modules.
2. app : contain code for main app and default circle.
3. plugins : This folder will contain codes related to extended plugins.

### App Module
Now let’s see how the module app is coded. This module contains the code for the main app and default circle.

![](/images/1_beW6RIQ-qJTkgSOEBso8ww.webp)

Here is how structure looks like,

![](/images/1_aj2gRikA9FZguijkOB2Ogw.webp)

Wait !!. Let me explain…

1. **app/library/plugins :**
This is the directory, where the app scans for plugins at runtime. Users can put jar files for preferred plugins into this directory. And those jar files will be read at runtime.

2. **pom.xml :**
The pom.xml is configured as below. It tells the app about itself and it’s parent.
```xml
<project
	xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         https://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>com.drawer.circle</groupId>
		<artifactId>circle_drawer</artifactId>
		<version>0.0.1</version>
	</parent>
	<groupId>com.drawer.circle.app</groupId>
	<artifactId>app</artifactId>
</project>
```
3. **app/src/main/java/c_interface/Circle.java :**

This is a interface which defined as below,
```java
package c_interface;
public interface Circle {
    public String getName();
    public void drawCircle(int radius);
}
```
This interface tells how the circle class should be structured. Both default circle class and plugin circle classes should implement this structure.
the getName method should return the name of the circle.
the drawCircle method should draw a circle when radius is given.

4. **app/src/main/java/c_default/BasicCircle.java :**

This class defines the logic of the basic circle as below.
```java
package c_default;

import java.awt.Graphics;
import javax.swing.JFrame;
import javax.swing.JPanel;
import c_interface.Circle;

public class BasicCircle implements Circle {

    public String getName() {
        return "basic";
    }

    public void drawCircle(final int radius) {
        JPanel pnlCircle = new JPanel() {
            public void paintComponent(Graphics g) {
                int X = 250 - radius;
                int Y = 250 - radius;
                g.drawOval(X, Y, radius * 2, radius * 2);
            }
        };
        JFrame frame = new JFrame();
        frame.add(pnlCircle);
        frame.setSize(500, 500);
        frame.setVisible(true);
    }
}
```
This circle class will draw a simple circle with black border, given the radius.

5. **app/src/main/java/Main.java :**

This is the Main class of the application. Main.java is defined as follows,
```java
import java.io.File;
import java.net.URL;
import java.net.URLClassLoader;
import java.util.HashMap;
import java.util.Scanner;
import c_default.BasicCircle;
import c_interface.Circle;
public class Main {
    static HashMap < String, Circle > supportedCircles = new
    HashMap < String, Circle > ();
    public static void main(String[] args) {
        loadDefaultSupportedCircles();
        loadPluginCircles();
        listAllSupportedCircle();
        drawCircle();
    }
    private static void loadDefaultSupportedCircles() {
        Circle basic = new BasicCircle();
        String name_of_basic = basic.getName();
        supportedCircles.put(name_of_basic, basic);
    }

    private static void loadPluginCircles() {
        try {
            File file = new File("./library/plugins/");
            File[] plugins = file.listFiles();
            for (File plugin_jar: plugins) {
                try {
                    URL url = plugin_jar.toURI().toURL();
                    URL[] urls = new URL[] {
                        url
                    };

                    ClassLoader cl = new URLClassLoader(urls);
                    Class < Circle > cls = 
                        (Class < Circle > ) cl.loadClass("plugins.PluginCircle");
                    Circle circle = cls.newInstance();
                    supportedCircles.put(circle.getName(), circle);
                } catch (Exception e) {
                    System.out.println("Error while adding plugin " + plugin_jar.getName());
                }
            }
        } catch (Exception e) {
            System.out.println("Adding Plugins Failed Due To : " + e.toString());
        }
    }

    private static void listAllSupportedCircle() {
        System.out.println("Supported Circle Types");
        System.out.println("======================");

        int i = 0;
        for (Object obj: supportedCircles.values().toArray()) {
            i++;
            Circle circle = (Circle) obj;
            System.out.println(i + ". " + circle.getName());
        }

    }

    private static void drawCircle() {
        Scanner scanner = new Scanner(System.in);

        System.out.print("\nEnter Circle Type : ");
        String user_pref = scanner.nextLine();

        System.out.print("Enter Circle Radius : ");
        int radius = scanner.nextInt();

        scanner.close();

        Circle user_pref_circle = supportedCircles.get(user_pref);

        if (user_pref_circle == null || radius <= 0) {
            System.out.println("Unsupported Circle Options");
            return;
        }

        user_pref_circle.drawCircle(radius);

    }
}
```
The main logic is divided into 4 main parts as shown below. Each part will be done by a specific method.
```java
  loadDefaultSupportedCircles();
  loadPluginCircles();
  listAllSupportedCircle();
  drawCircle();
```
* loadDefaultSupportedCircles() : This will load and store the default Circle type supported by the app. It will be loaded as an instance and put into a hashmap along with the name.

* loadPluginCircles() : This will scan ./library/plugins/ directory and pick plugin jar files. Each jar file will be loaded as an instance and put into a hashmap along with the name.

* listAllSupportedCircle() : This method will indicate all the circle types supported by the app (default + plugin). Then the user can pick up one of them.

* drawCircle(): This method is used to get the user preferred circle type and radius. After user inputs it will draw the circle on JFrame.

## Testing

Now let’s test the application. Here is the outcome of the application built so far.

![](/images/1_mRgHQ3lqKKMgsp1mZxnRfA.webp)

Since No plugins are added, It supports only basic type of circle. Here is the circle drawn.

![](/images/1_aKXk2aDOZjP-IkGmLujtMQ.webp)

It’s all for part 1. Rest will be continued in [Part 2](/docs/java/plugin-development/part-2/). In that, I hope to add two more circles as the plugins.

Happy Coding 🙌