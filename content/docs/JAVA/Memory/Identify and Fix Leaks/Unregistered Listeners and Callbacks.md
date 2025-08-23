---
weight: 500
---

# Java Memory Leaks : Unregistered Listeners and Callbacks

## Problem

In Java GUI-based applications, it is common to use observers and observables. However, the observers registered with an observer should be carefully unregistered when no longer in use. Otherwise, these observer objects won’t be collected by the Garbage Collector, even though that is no longer in use, which can lead to a memory leak in Java.

To simulate this situation, let's build a simple swing UI as shown in the code below.

```java
import javax.swing.*;
import java.awt.*;
import java.lang.management.ManagementFactory;
import java.util.List;
import java.util.*;
import java.util.Timer;
```
```java
public class SystemUsageObserverApp {

    public static void main(String[] args) throws InterruptedException {
        Thread.sleep(10000);
        SwingUtilities.invokeLater(() - > new SystemUsageObserverApp().createAndShowGUI());
    }

    private final SystemUsageObservable systemUsageObservable = new SystemUsageObservable();
    private boolean isFloat = false;

    private void createAndShowGUI() {
        JFrame frame = new JFrame("System Usage Observer (Swing)");
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        frame.setSize(400, 800);

        JPanel panel = new JPanel();
        panel.setLayout(new BoxLayout(panel, BoxLayout.Y_AXIS));

        drawPanel(panel);

        frame.setContentPane(new JScrollPane(panel));
        frame.setVisible(true);

        startSystemUsageUpdates();
    }

    private void drawPanel(JPanel panel) {
        JButton reloadButton = new JButton(isFloat ? "Change to Percent" : "Change to Float");
        reloadButton.addActionListener(e - > reloadPanel(panel));
        panel.add(reloadButton);

        systemUsageObservable.setFload(isFloat);
        for (int i = 0; i < 10000; i++) {
            JLabel label = new JLabel();
            label.setFont(new Font("Monospaced", Font.PLAIN, 12));
            systemUsageObservable.add(label);
            panel.add(label);
        }
    }

    private void reloadPanel(JPanel panel) {
        isFloat = !isFloat;
        panel.removeAll();
        panel.revalidate();
        drawPanel(panel);
        panel.repaint();
        System.gc();
    }

    private void startSystemUsageUpdates() {
        Timer timer = new Timer(true);
        timer.scheduleAtFixedRate(new TimerTask() {
            @Override
            public void run() {
                com.sun.management.OperatingSystemMXBean osBean = (com.sun.management.OperatingSystemMXBean) ManagementFactory
                    .getOperatingSystemMXBean();
                double usage = osBean.getSystemCpuLoad();
                System.out.println("Usage : " + usage);
                systemUsageObservable.setSystemUsage(usage);
            }
        }, 0, 1000);
    }
}
```
```java
class SystemUsageObservable {
    private double systemUsage;
    private List < JLabel > observerLabels = Collections.synchronizedList(new ArrayList < > ());
    private boolean isFloat = false;

    public void add(JLabel observerLabel) {
        this.observerLabels.add(observerLabel);
    }

    public void removeAllListners() {
        observerLabels.clear();
    }

    public void setFload(boolean isFloat) {
        this.isFloat = isFloat;
    }

    public void setSystemUsage(double usage) {
        this.systemUsage = usage;
        notifyObservers(usage);
    }

    private void notifyObservers(double usage) {
        System.out.println("Notifying Observers : " + observerLabels.size());
        observerLabels.forEach((label) - > {
            SwingUtilities.invokeLater(() - > label.setText(isFloat ? String.format("System Usage: %.5f", usage) :
                String.format("System Usage: %.2f%%", usage * 100)));
        });
    }

    public double getSystemUsage() {
        return systemUsage;
    }
}
```

In here, when UI is initially loaded/gets reloaded. We add labels to the panel, which will show the current CPU usage value. These labels will be registered as observers into the Observable class called `SystemUsageObservable`, which is responsible for notifying labels about updated CPU usage values. The CPU usage value will be synced into `SystemUsageObservable` every 1 second and we have a button to change between `Float Mode` and `Percent Mode`. 

Each time this button is clicked, it redraws the panel with a new set of labels and buttons to represent the relevant format. Now let's run the simulation. Assume we are changing `FLOAT MODE` <-> `PERCENT MODE` frequently over time. We are getting the following behaviour.
```text
-	Memory Usage Becomes Near 200 MB
-	UI Got Unresponsive
-	Observable still notifies non-rendered labels too
```

![](/images/unreleased_listners_1.png)

## Cause

*As we can see, the memory usage rises every time we click the button. What happened?*

```java
private void drawPanel(JPanel panel) {
    JButton reloadButton = new JButton(isFloat ? "Change to Percent" : "Change to Float");
    reloadButton.addActionListener(e - > reloadPanel(panel));
    panel.add(reloadButton);

    systemUsageObservable.setFload(isFloat);
    for (int i = 0; i < 10000; i++) {
        JLabel label = new JLabel();
        label.setFont(new Font("Monospaced", Font.PLAIN, 12));
        systemUsageObservable.add(label);
        panel.add(label);
    }
}
```

The issue is in the above code for `drawPanel`. Each time we click the button, it clears the panel and redraws it. But what we **missed was unregistering/cleaning up unused non-rendered labels**, which still listen to observer notifications. 

## Fix

As a fix, we have now updated the code to fix this by unregistering invalidated labels from the Observable before the redraw panel.

```java
private void drawPanel(JPanel panel) {
    systemUsageObservable.removeAllListners(); // UNREGISTER UNUSED ELEMENTS PROPERLY
    JButton reloadButton = new JButton(isFloat ? "Change to Percent" : "Change to Float");
    reloadButton.addActionListener(e - > reloadPanel(panel));
    panel.add(reloadButton);

    systemUsageObservable.setFload(isFloat);
    for (int i = 0; i < 10000; i++) {
        JLabel label = new JLabel();
        label.setFont(new Font("Monospaced", Font.PLAIN, 12));
        systemUsageObservable.add(label);
        panel.add(label);
    }
}
```

Now let's see how things work. 

![](/images/unreleased_listners_2.png)

As we can see now, we see a major improvement in memory usage and UI responsiveness
```text
-	Memory Usage becomes less than 100 MB
-	UI is Responsive with even 10,000 labels
-	Observable notifies rendered labels only
```

As we can see, To fix these kind of memory leaks, unused observers no longer used, should be properly cleaned up to avoid a memory leak in Java.

Happy Coding 🙌