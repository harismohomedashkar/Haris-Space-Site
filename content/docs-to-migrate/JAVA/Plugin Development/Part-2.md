---
weight: 200
---
# How to implement plugin support for a Java Application : Part 2

![](/images/0_acWCYTTrqX1RY1FN.webp)

## Introduction

As I mentioned in the [Part 1](/docs/java/plugin-development/part-1/), in this article I will implement two plugins to draw two different types of circles. Here are the two circle types, that will be added,

1. **redcircle** : This is a circle which has a red colored border. And this circle is filled with red color.

2. **bluebordercircle** : This circle is not filled with any color. However this circle contains a blue colored border.

Without wasting time, let’s move into code.

## Implementation

I am going to start with the redcircle.

### redcircle
#### Step 1: Create Maven Module

As the first step I have created a maven module named *red_circle* in *circle_drawer/plugins/* directory. The structure of the module is shown below,

![](/images/1_hRvgryU3SKwgszvsHeMPeQ.webp)

#### Step 2: Define Parent

As the next step I defined *circle_drawer* as the parent of this *red_circle* module. After that *pom.xml* was looked like below,

```xml
<project
	xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema\-instance" 
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         https://maven.apache.org/xsd/maven\-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>com.drawer.circle</groupId>
		<artifactId>circle_drawer</artifactId>
		<version>0.0.1</version>
		<relativePath>../../pom.xml</relativePath>
	</parent>
	<groupId>com.drawer.circle.plugins.red_circle</groupId>
	<artifactId>red_circle</artifactId>
</project>
```

#### Step 3: Add red_circle module to circle_drawer

As the next step I wanted to add the *red\_circle* module into the *circle\_drawer* parent project. Even if *red\_circle* resides in *circle\_drawer*, *circle\_drawer* doesn’t know about *red\_circle*. Hence, I have to introduce the *red\_circle* module to *circle\_drawer* by adding it to *pom.xml*. After doing that, the *pom.xml* of *circle\_drawer* looked like below.

```xml
<project
	xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema\-instance" 
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         https://maven.apache.org/xsd/maven\-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.drawer.circle</groupId>
	<artifactId>circle_drawer</artifactId>
	<version>0.0.1</version>
	<packaging>pom</packaging>
	<modules>
		<module>app</module>
		<module>plugins/red_circle</module>
	</modules>
</project>
```

#### Step 4: Let’s Try to build the project

As the next step I am going to build the project using the maven build command.
I am using eclipse IDE. For build I had to right click on *circle\_drawer*, and go to *Run As\-> Maven Build*. I used a clean package as the goal. And then clicked Run. After successful built, output was as below,

![](/images/1_17gCrvpFsOCxsmLv7I4MuA.webp)

#### Step 5: Add app as dependency

As I mentioned before, all plugins should use the *Circle.java* interface when implementing plugins. Since the *Circle.java* resides in the app. We should add app as a dependency. For this I have to add the following lines into the *pom.xml* of *red\_circle*.

```xml
<dependencies>
	<dependency>
		<groupId>com.drawer.circle.app</groupId>
		<artifactId>app</artifactId>
		<version>0.0.1</version>
	</dependency>
</dependencies>
```

Then final *pom.xml* of *red\_circle* was as below,

```xml
<project
	xmlns="http://maven.apache.org/POM/4.0.0"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema\-instance" 
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         https://maven.apache.org/xsd/maven\-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<parent>
		<groupId>com.drawer.circle</groupId>
		<artifactId>circle_drawer</artifactId>
		<version>0.0.1</version>
		<relativePath>../../pom.xml</relativePath>
	</parent>
	<groupId>com.drawer.circle.plugins.red_circle</groupId>
	<artifactId>red_circle</artifactId>
	<dependencies>
		<dependency>
			<groupId>com.drawer.circle.app</groupId>
			<artifactId>app</artifactId>
			<version>0.0.1</version>
		</dependency>
	</dependencies>
</project>
```

#### Step 6: Coding plugins.PluginCircle.java class

As the next step I am going to implement the logic for the drawing circle. For this first I need to create a package named plugins in *src/main/java* of *red\_circle*. Then I created a *PluginCircle.java* class in the plugins package. This class implements the *Circle* interface. After all, I just had to override the *getName\(\)* and *drawCircle\(\)* methods. Final class is shown below,

![](/images/1_iADPmFnxjz0lzcR-YDl71Q.webp)

```java
package plugins;
import java.awt.Color;
import java.awt.Graphics;
import javax.swing.JFrame;
import javax.swing.JPanel;
import c_interface.Circle;
public class PluginCircle implements Circle {
    public String getName() {
        return "redcircle";
	}
    public void drawCircle(final int radius) {
        JPanel pnlCircle = new JPanel() {
            public void paintComponent(Graphics g) {
                int X = 250 - radius;
                int Y = 250 - radius;
                g.setColor(Color.red);
                g.fillOval(X, Y, radius * 2, radius * 2);
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

#### Step 7 : Rebuild

As the step 7 rebuild the maven project *circle\_drawer* as shown in step 4. Final built result looked like below,

![](/images/1_An6WJ3KOlnf6MYdnrq84FA.webp)

#### Step 8 : copy jars into plugin directory

Now just go to the target directory of *red\_circle* and copy the jar file. Now paste that jar file into *circle\_drawer/app/library/plugins* as shown below,

![](/images/1_L3EdAmbX-5oBRuR4HmRn6A.webp)

That’s all.. Now let’s test

#### Testing

Now just run the app module. We can see that now it supports two circle types. Since we tried basic circle in previous article, this time I am going to try redcircle, which we added through plugin. Output will be as below.

![](/images/1_nnl28EPVIsc7e6u41A7_OQ.webp)

![](/images/1_7I2yBclMlhh-kxgAPbhJIA1.webp)

### bluebordercircle

I am not going to describe all the steps since they are the same as *red\_circle*. Code for the PluginCircle.java will be as below,

```java
package plugins;
import java.awt.Color;
import java.awt.Graphics;
import javax.swing.JFrame;
import javax.swing.JPanel;
import c_interface.Circle;
public class PluginCircle implements Circle {
    public String getName() {
        return "bluebordercircle";
    }
    public void drawCircle(final int radius) {
        JPanel pnlCircle = new JPanel() {
            public void paintComponent(Graphics g) {
                int X = 250 - radius;
                int Y = 250 - radius;
                g.setColor(Color.BLUE);
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

After adding jar file of bluebordercircle, the output of app module will be as follow,

![](/images/1_0nSFHyK9sPF6lrpQ9skrBw.webp)

![](/images/1_njzdW0j2VhZrxtC4vxKCIw.webp)

As shown in both articles. The circle drawer java application only supports basic circle type. And we can add any number of new circle types by introducing new plugins.

Finally we were able to build an expandable circle drawer due to this plugin architecture. I hope that you could able to gain some precious knowledge after reading both articles. 

Thank You.\
Happy Coding 🙌
