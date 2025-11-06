---
weight: 200
---

# Java Memory Leaks : Static Fields

As developers, we should be more careful when using static fields. Unlike non-static fields (which have a shorter life span), static fields are loaded with the class and exist until the class gets unloaded, which can exist until the complete program ends. Due to this, *we should only mark really static things as static fields*.

Let's focus on an example. Assume we are developing an online fraud detection application, and we have a file named blacklisted_ips (containing 1,000,000 IP addresses), which we take from a text file in S3. Every time we check IP validation, we have to compare it with this file, which makes the program a bit slow. As a solution to this, assume we are loading these IP addresses into a static list at the very beginning of the program, as shown in the code below (Not a best practice, also IP list can change dynamically, Just using as an example for simulation).

*StaticFieldSimulation.java*
```java
public class StaticFieldSimulation {

    public static void main(String[] args) throws InterruptedException {

        Thread.sleep(10000);
        System.out.println("IPValidatior class not loaded yet");
        Thread.sleep(10000);
        System.out.println("IPValidatior class not loaded yet even after 10s");

        System.out.println("IPValidatior class will be loaded now");

        IPValidatior ipValidatior1 = new IPValidatior("192.168.8.1"); // Valid IP
        System.out.println(String.format("IsValid : " + ipValidatior1.isValid()));
        System.gc();
        Thread.sleep(5000);
        System.out.println("IPValidatior static field is not collected by Garbage Collector, even not in use");

        IPValidatior ipValidatior2 = new IPValidatior("192.168.8..1"); // Invalid IP
        System.out.println(String.format("IsValid : " + ipValidatior2.isValid()));
        System.gc();
        Thread.sleep(5000);
        System.out.println("IPValidatior static field is not collected by Garbage Collector, even not in use");

        IPValidatior ipValidatior3 = new IPValidatior("122.131.44.253"); // Black Listed IP
        System.out.println(String.format("IsValid : " + ipValidatior3.isValid()));
        System.gc();
        Thread.sleep(5000);
        System.out.println("IPValidatior static field is not collected by Garbage Collector, even not in use");
        System.out.println("Static field will be collected, since class will be unloaded now");
    }

}
```

*IPValidatior.java*
```java
class IPValidatior {

    private static List < String > blackListedIps;
    private static final Pattern IPV4_PATTERN = Pattern.compile("^((25[0-5]|2[0-4]\\d|1\\d{2}|[1-9]?\\d)(\\.|$)){4}$");
    private String Ip;

    static {
        blackListedIps = S3Manager.loadBlackListedIpArrayFromS3();
    }

    public IPValidatior(String Ip) {
        this.Ip = Ip;
    }

    public boolean isValid() {
        // IP format validation
        if (Ip == null || !IPV4_PATTERN.matcher(Ip).matches()) {
            return false;
        }

        // Make Sure Not Black Listed
        return !blackListedIps.contains(Ip);
    }

}
```
> Note - Used Visual VM to debug memory

Now, as shown below, when class get intially loaded into the program, this list will be loaded into the static field, and it will reside there until a specific class gets unloaded or an Object gets dereferenced. Garbage collectors won’t collect it even though this field is not in use, which leads to inefficient memory usage.

![](/images/static-fields.png)

In summary, once an Object gets loaded into a static field in the application, it will reside in memory until the class gets unloaded / Object gets dereferenced; even though it is not being used anymore, the garbage collector will not collect it. Due to this, we should be more careful when working with static fields, regarding their memory consumption throughout the application.

Happy Coding 🙌