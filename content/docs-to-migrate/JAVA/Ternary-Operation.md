# Java : Ternary Operation

![](/images/0_AmLgBqUJfksEgPbv.webp)

## Introduction

In this article, we are going to discuss a quick and efficient operation used in most programming languages. This operation is known as a Ternary Operation.

We are commonly using if, else, else if, switch case operation to handle our conditional tasks. Ternary operation is also one of the *conditional operations* introduced by programming languages.

## Structure

First of all, I would like to introduce the structure of this operator in java.

```java
variable var1 = (condition) ? (value If True):(value If False)
```

This operator has three main parts. Those are,
```text
Variable
Condition
Return Values
```

*When we run this operator first it checks whether the given condition is true. If true it stores Value If True in var1. If the given condition is false it stores Value If False in var1. Below java code will show how this works.*

## Code Example 1
```java
class Ternary{
    public static void main(String args[]) {
        int score;
        String result;        
        score=75;
        result = (score>=50) ? ("Passed") : ("Failed");
        System.out.println(result);
        score=30;
        result = (score>=50) ? ("Passed") : ("Failed");
        System.out.println(result);
    }
}
```
Output:
```text
Passed
Failed
```
Now you have the question: *“Why do we need this operation? We can use an “If” Statement instead of this”*.\
Yes, you are correct, we can use an “if” condition instead of using ternary operation. But after reading the below example I’m sure that you will choose ternary over if.

## Code Example 2

### Using If
Grading Students Mark Using If, else, else if Condition :
```java
class Ternary{
    public static void main(String args[]) {
        int score;
        String result;        
        score=67;
        if(score>=75){
            result = "A";
        }
        else if(score>=65){
            result = "B";
        }
        else if(score>=55){
            result = "C";
        }
        else if(score>=40){
            result = "S";
        }
        else{
            result = "F";
        }        
        System.out.println(result);
    }
}
```
It takes 23 code lines \!\!\!

### Using Ternary
Grading Students Mark Using Ternary Operation :
```java
class Ternary{
    public static void main(String args[]) {
        int score;
        String result;
        score=30;
        result=(score>=75)?"A":
               (score>=65)?"B":
               (score>=55)?"C":
               (score>=40)?"S":"W";
        System.out.println(result);
    }
}
```
Now it took 12 code lines \!\!\!\
Now you can see that codes can be shrunk using ternary operation.
## Conclusion
Line by line we came to the end of the article. In this we discussed,
```text
Introduction to Ternary
Structure of Ternary
Main Parts Ternary
Sample code using Ternary
Comparing If and Ternary.
```

I hope that this article was helpful to you.\
Thank you for reading.

Happy Coding 🙌
