# Spring Boot : Aspect Oriented Programming (AOP)

![](/images/Screenshot%202025-03-10%20110544.png)

## Problem
Let's begin with an example.

Assume we have a system where some methods are only accessible based on some user type \(ADMIN, USER, GUEST\) and access mode \(API, PORTAL\).
Basic code will look like this.

```java
@Service

class OrderManagerService{

    void submitOrder(String orderId){

	    // User Type Validation Logic -- 10 Lines
	    // Access Mode Validation Logic -- 8 Lines
        // Log Access Record -- 2 Lines

        
        // BUSSINESS LOGIC FOR ORDER SUBMISSION -- 20 Lines


        // Log Time Took - 3 Lines
        // Log Success or Error - 2 Lines

    }



    void cancelOrder(String orderId){

	    // User Type Validation Logic -- 10 Lines
	    // Access Mode Validation Logic -- 8 Lines
        // Log Access Record -- 2 Lines

        
        // BUSSINESS LOGIC FOR CANCEL ORDER -- 20 Lines


        // Log Time Took - 3 Lines
        // Log Success or Error - 2 Lines

    }

}

... 15 more methods like this.
```

As we can see there is a repetitive set of codes, which is not related to business logic. Due to these code lines,

* There will be lot of duplications
* Even if small change needed, we have to change it every where, which makes hard to maintain
* Code becomes complex

These types of codes we used for logging, security, and transaction management are called as ***cross\-cutting concerns***, because those are *repetitively applied for methods across the application*. Hence the best approach is to separate these code lines in a separate common class, this is where we need ***Aspect Oriented Programming***. Now let's see how we can apply AOP for above example. 

## Fixing With AOP

### Add Dependency

Initially we have to add required dependency.
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

### Create Custom Annotation

Then we have to create some identifier, which we can use to mark the locations where these logics needs to be applied. In this example we use annotation interface for this.

```java
import java.lang.annotation.*;


@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME) < -- -- -- -- -- -- --( * )[6]
public @interface ValidateRequest { < -- -- -- -- -- -- --( * )[1]

    RequestType[] types();

    UserType[] allowedUserTypes();

}

enum RequestType {
    API,
    PORTAL
}

enum UserType {
    ADMIN,
    USER,
    GUEST
}
```

### Replace Repetitive Logic

Now we remove all repetitive logics and apply this annotation to methods we need. Code will look like as follows,

```java
@Service
class OrderManagerServcie { < -- -- -- -- -- -- --( * )[2]

    [1]
    @ValidateRequest(types = {
        RequestType.PORTAL,
        RequestType.API
    }, allowedUserTypes = {
        UserType.USER
    })
    void submitOrder(String orderId) { < -- -- -- -- -- -- --( * )[3]

        // BUSSINESS LOGIC FOR ORDER SUBMISSION -- 20 Lines

    }


    [1]
    @ValidateRequest(types = {
        RequestType.PORTAL
    }, allowedUserTypes = {
        UserType.ADMIN,
        UserType.USER
    })
    void cancelOrder(String orderId) { < -- -- -- -- -- -- --( * )[3]

        // BUSSINESS LOGIC FOR CANCEL ORDER -- 20 Lines

    }

}
```

### Create Aspect

Now we have to create Aspect with repetitive logics. For this we use @Aspect annotation.
```java
@Aspect
@Component
public class ValidateRequestAspect { < -- -- -- -- -- -- --( * )[4]

    @Pointcut("@annotation(validateRequest)") < -- -- -- -- -- -- --( * )[1]
    public void validateRequestPointcut(ValidateRequest validateRequest) {}

    // Before Advice
    @Before("validateRequestPointcut(validateRequest)") < -- -- -- -- -- -- --( * )[5]
    public void beforeAdvice(ValidateRequest validateRequest) {
        System.out.println("Before Advice: Checking request type and user type...");
    }

    // After Advice
    @After("validateRequestPointcut(validateRequest)") < -- -- -- -- -- -- --( * )[5]
    public void afterAdvice(ValidateRequest validateRequest) {
        System.out.println("After Advice: Logging request details...");
    }

    // After Returning Advice
    @AfterReturning("validateRequestPointcut(validateRequest)") < -- -- -- -- -- -- --( * )[5]
    public void afterReturningAdvice(ValidateRequest validateRequest) {
        System.out.println("After Returning Advice: Successfully executed method.");
    }

    // After Throwing Advice
    @AfterThrowing(value = "validateRequestPointcut(validateRequest)", throwing = "ex") < -- -- -- -- -- -- --( * )[5]
    public void afterThrowingAdvice(ValidateRequest validateRequest, Exception ex) {
        System.out.println("After Throwing Advice: Exception occurred - " + ex.getMessage());
    }

    // Around Advice
    @Around("validateRequestPointcut(validateRequest)") < -- -- -- -- -- -- --( * )[5]
    public Object aroundAdvice(ProceedingJoinPoint joinPoint, ValidateRequest validateRequest) throws Throwable {
        System.out.println("Around Advice: Before proceeding...");
        Object result = joinPoint.proceed(); // Let's Execute Method and Get Results
        System.out.println("Around Advice: After proceeding...");
        return result;
    }

}
```

## Spring AspectJ Concepts

As we can see now we have successfully moved repetitive logics to common aspect, where we can centrally manage cross cutting concerns more effectively.

Now let's identify each components we numbered one by one 


1. ***Pointcut*** : Identifier we used to tell aspects, where it should consider apply advices. For example in our example we say we have to apply advces where we applied *@ValidateRequest* annotation.
2. ***Target Objects*** : Objects whose methods are inteferred by Aspect advices. In our example one will be *OrderManagerServcie* Object.
3. ***Join Point*** : Execution point where aspect advices are being applied. In our example those are  *submitOrder, cancelOrder, ...* methods.
4. ***Aspect*** : The component, where we define pointcut and the advices that should be applied on identified joinpoints. In our example it will be *ValidateRequestAspect* class.
5.  ***Advice*** : Methods we use in aspect to define what should happen on those joinpoints. There are 5 types of advices we can define.

    1. Before Advice: Runs before method execution
    2. After Advice: Runs after method execution
    3. After Returning Advice: Runs if the method completes successfully
    4. After Throwing Advice: Runs if an exception occurs
    5. Around Advice: Runs before & after execution

6. ***Weaving*** : Here we define retention of the annotation. And it's marked as *RUNTIME* since we are using @ValidateRequest in an Aspect, it will likely be checked at runtime to apply role\-based authorization. Without *RUNTIME*, the aspect will not be able to read the annotation, making it ineffective.

That's all about AOP for now.

Happy Coding 🙌
