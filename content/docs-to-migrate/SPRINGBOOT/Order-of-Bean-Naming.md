# Spring Boot : Order of Bean Naming
![](/images/Screenshot%202025-03-11%20113739.png)

In this article, we will explore how Spring Boot resolves ambiguity and determines bean names in the application context. Spring Boot follows these rules in order to determine the bean names.

## Name in @Bean Annotation

First Spring Boot checks whether bean name is defined in @Bean annotation as follows. If so it will be used as Bean name to register in the context.

```java
    @Bean(name = "myDemoService")
    public MyService initMyService(){
        return new MyService();
    }
```
Bean Name : *myDemoService*

## Name in Component Annotations

If not, Spring Boot checks whether bean name is defined in annotation like @Component, @Service, etc. as follows. If so it will be used as Bean name to register in the context.

```java
@Service("sampleService")
class MyService {
    
}
```
Bean Name : *sampleService*

## Bean Configured Method Name

If not, Spring Boot checks whether object is initialized through @Bean \(this time no explicit name within bean annotation given\) method, and use that method name as Bean name to register in the context.
```java
    @Bean
    public MyService mySimpleService(){
        return new MyService();
    }
```
Bean Name : *mySimpleService*

## Class First Letter Lowercased

If not, Spring Boot uses class name with first letter lowercased as Bean name in the context.

```java
@Service
class EmailService {

}
```
Bean Name : *emailService*

## Fully Qualified Name
When using class name there can be some classes with same name but in different packages. For example.
```java
com.example.packageone.EmailService
com.example.packagetwo.EmailService
```
In that case it uses Fully Qualified Names as bean name to resolve conflicts as follows,
Bean Names : *com.example.packageone.EmailService, com.example.packagetwo.EmailService*

> Note : This is just an general order of Bean naming. There are some special cases where naming and its order can be different than above.

Happy Coding 🙌
