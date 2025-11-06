---
weight: 300
---
# GOF Design Patterns : Abstract Factory

![](/images/Screenshot%202025-02-12%20110747.png)

## Introduction

The Factory Method is a creational design pattern that provides an interface for creating single objects. The Abstract Factory is also a creational design pattern, but it focuses on creating factories that produce families of related objects. In other words, an Abstract Factory is a factory of factories \(a super factory\).

## Problem

Let's look at an example.

In this example, we look into UI building logic. Let's say we want to create Buttons and Checkboxes for Windows and Mac.
So, the objects are as follows:
```java
interface Button { void renderButton(); }
interface Checkbox { void renderCheckbox(); }

class WindowsButton implements Button {
    public void renderButton() {
        System.out.println("Rendering Windows Button");
    }
}

class MacButton implements Button {
    public void renderButton() {
        System.out.println("Rendering Mac Button");
    }
}

class WindowsCheckbox implements Checkbox {
    public void renderCheckbox() {
        System.out.println("Rendering Windows Checkbox");
    }
}

class MacCheckbox implements Checkbox {
    public void renderCheckbox() {
        System.out.println("Rendering Mac Checkbox");
    }
}
```

Now, let's write code to initialize these elements and render a frame for both Windows and Mac.
```java
class UI {
    public void windowsFrame() {
        Button winButton = new WindowsButton();
        Checkbox winCheckBox = new WindowsCheckbox();
        winButton.renderButton();
        winCheckBox.renderCheckbox();
    }

    public void macFrame() {
        Button macButton = new MacButton();
        Checkbox macCheckBox = new MacCheckbox();
        macButton.renderButton();
        macCheckBox.renderCheckbox();
    }
}
```
If we observe the code, we are tightly coupling the UI with the OS \(Windows/Mac\). If we want to extend Linux support, we have to add a completely new, repetitive method named linuxFrame, which also violates SOLID principles.

## Fix It

What if we use a higher\-level factory that generates Windows/Mac/Linux \(or whatever we support\), handles element generation logic, and injects it into the UI at runtime based on the detected OS?

This is where the Abstract Factory comes into play. Let's implement it in this example.
```java
interface UIFactory {
    Button createButton();
    Checkbox createCheckbox();
}

class WindowsUIFactory implements UIFactory {
    public Button createButton() {
        return new WindowsButton();
    }

    public Checkbox createCheckbox() {
        return new WindowsCheckbox();
    }
}

class MacUIFactory implements UIFactory {
    public Button createButton() {
        return new MacButton();
    }

    public Checkbox createCheckbox() {
        return new MacCheckbox();
    }
}
```

As we can see, we now have higher\-level UI factories responsible for handling OS\-specific elements.

Now, we can decouple the UI factory from OS\-based logic as follows:

```java
class UI {
    UIFactory uiFactory;

    public UI(UIFactory uiFactory) {
        this.uiFactory = uiFactory;
    }

    public void generateFrame() {
        Checkbox checkBoxForAccept = uiFactory.createCheckbox();
        Button buttonForSubmit = uiFactory.createButton();
        checkBoxForAccept.renderCheckbox();
        buttonForSubmit.renderButton();
    }
}

class AbstractFactoryExample {
    public static void main(String[] args) {
        UI windowsUI = new UI(new WindowsUIFactory());
        windowsUI.generateFrame();

        UI macUI = new UI(new MacUIFactory());
        macUI.generateFrame();
    }
}
```

Output:
```text
Rendering Windows Checkbox  
Rendering Windows Button  
Rendering Mac Checkbox  
Rendering Mac Button 
```

## When to use ?

- When we want to generate families of grouped elements \(Windows element family, Mac element family\).
- To decouple the generation of families of elements from the implementation logic.

## Compare With Factory-Method

- Factory Method : A method that returns an instance of a single product.
- Abstract Factory : A factory that returns multiple related products.

## Class Diagram

![](/images/abstract_factory.drawio.png)

Happy Coding 🙌
