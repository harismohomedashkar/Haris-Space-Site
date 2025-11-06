---
# 786
weight: 700
---
# DB Transactions : Propagation Levels

## Introduction

Let's assume we have a method which should run as a transaction and within that method there are several subprocesses that should execute as transactions. Simply put, we can see multiple nested transactions within a single transactional context.

The consideration is how we should handle such things,
- Do we need to run everything (root + nested) as one single transaction ? **OR**
- Do we need to consider each nested transaction as separate new transaction ? **OR**
- Any Otherway ?

This is where **Transaction Propagation Levels** comes into discussion.

By selecting a suitable transaction propagation level, we can instruct program how to handle multiple transaction invocations that exists within a single transaction context.

There are 5 main propagation levels we can use. Those are

1. REQUIRED (default)
2. REQUIRES_NEW
3. SUPPORTS
4. NOT_SUPPORTED
5. MANDATORY

Now let's discuss each one in detail.

## Required

- This is the ***default***.
- If this transaction is invoked within an existing transactional context, it will join the existing transaction.
- If this is invoked outside of a transactional context, it will create a new transaction for itself.
  

Note: Following is a simplified example code

```java
@Transactional(propagation = Propagation.REQUIRED)
public void transfer(TransactionDTO transactionDto){
    String senderAccountNo = transactionDto.getSenderAccountNo()
    String receiverAccountNo = ...
    Double amount = ...
}

@Transactional(propagation = Propagation.REQUIRED)
public void multipleTransfer(List<TransactionDTO> transactionDtos){
    for(TransactionDTO transactionDto: transactionDtos){
        transfer(transactionDto)
    }
}
```

Means if we call `multipleTransfer` method, the `transfer` method will just join existing transaction created by `multipleTransfer` (if single transfer failed, the complete multipleTransfer transaction will be rolled back).

If we directly called the `transfer` method, it will create a transaction for itself.

## Requires_new

- If this is invoked within an existing transactional context, it will halt the existing transaction, and this method will create a new transaction and run in that new context, Eventually, the existing transaction can continue after completion.
- If this is invoked outside an existing transaction, it will create a new transaction for itself.

> In Summary: Regardless of being invoked within transactional context or not, this will create a new transaction for itself and operates in that separate transactional context.

Note: Following is simplified example code

```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void transfer(TransactionDTO transactionDto){
    String senderAccountNo = transactionDto.getSenderAccountNo()
    String receiverAccountNo = ...
    Double amount = ...
}

@Transactional(propagation = Propagation.REQUIRED)
public void multipleTransfer(List<TransactionDTO> transactionDtos){
    for(TransactionDTO transactionDto: transactionDtos){
        transfer(transactionDto)
    }
}
```

Means if we call `multipleTransfer` method, when `transfer` method is being executed, `multipleTransfer` transaction will be paused, the `transfer` method will create a new transaction for itself and process in that new context, after completion of that, then `multipleTransfer` transaction will continue. (if a single transfer failed, only that will be rolled back, multipleTransfer transactions will continue to proceed )

If we directly called `transfer` method, it will create a transaction for itself.


## Supports

- If this is invoked within an existing transactional context, it will join the existing transaction.
- If this is invoked without transactional context, it will execute non-transactionally.

Note: Following is a simplified example code

```java
@Transactional(propagation = Propagation.SUPPORTS)
public void readUsers(){
    // 
}
```

This makes sure in a transactional context it joins existing transaction, and in a non-transactional context it will execute non transactionally. This will be useful when method logical behaviour depends on the transactional context it runs in.

## Not_supported

This make sure method will be executed non transactionally, regardless of whether it run in transactional context or not. 

- Even if this is invoked within an existing transactional context, method will run as non-transactional.
- If this is invoked without a transactional context, the method will run as non-transactional.

Note: Following is simplified example code

```java
@Transactional(propagation = Propagation.NOT_SUPPORTED)
public void addLog(String msg){
    // 
}
```
Assume we want to keep this logs for audit purpose, regardless of main transaction failed or not. Then we should use `NOT_SUPPORTED` transaction level to exclude this operation from the main transactional context.

If this method is called within an exsing transactional context, that transaction will be suspended until the method completes, and will be resumed after method executed.
This will be useful when we need to run some method in a non-transactional manner, even in a transactional context.

Like we want to prevent that operation from getting rolled back when main transaction failed and rolled back.

## Mandatory

- The method should be invoked within an active transactional context.
- If active transactional context found, this will join it. 
- If no existing transactional context found, then the method will throw an exception `TransactionRequiredException`.

Note: Following is a simplified example code

```java
@Transactional(propagation = Propagation.MANDATORY)
public void updateBalance(String accountNo, double amount){
    // 
}
```

Assume we have a helper method to update the balance. But this method should be called within an existing transactional context to make sure it is running in a meaningful context with other operations, which is part of a larger transaction. 

And this method itself running without transactional context is meaningless, (we can't update the balance directly, there should be some other place where this balance change comes from).

We can set this `MANDATORY` propagation level on the helper function, that is meaningless to run alone, but it should be run within existing transaction as a singe operation in a meaningful context.

## Summary

Here is summary of what we discussed so far,

| Propagation Level | Without Transactional Context | Within Transactional Context                           |
|-------------------|-------------------------------|--------------------------------------------------------|
| REQUIRED          | Creates New for Itself        | Will Join Existing                                     |
| REQUIRES_NEW      | Creates New for Itself        | Suspend Existing, Creates New for Itself               |
| SUPPORTS          | Run Without Transaction       | Will Join Existing                                     |
| NOT_SUPPORTED     | Run Without Transaction       | Suspend Existing, Run Outside of Transactional Context |
| MANDATORY         | Will Throw An Error           | Will Join Existing                                     |



Yeah, that's all for propagation levels for now\
Hope this article was helpful\
Thank You!\
Happy Coding 🙌