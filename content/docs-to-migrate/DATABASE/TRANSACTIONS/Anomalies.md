---
# 786
weight: 500
---
# DB Transactions : Anomalies
## Introduction

In [ACID](/docs/database/transactions/acid/) we already discussed the ACID Properties. There we discussed Isolation ("**I**" in ACID) briefly. However in this article we will discuss Isolation indepth. 

First let's see what Isolation in Database Transaction means. As we discussed already,

> Isolation property in transaction, isolates a transaction from other transactions, and enforces limitations on what it can see of changes made by other transactions. 

Based on the use cases, developers can select different isolation levels to make sure that, multiple transactions can occur concurrently without leading the database state into inconsistency eventually. 

We will discuss, different Isolation levels supported by modern Databases. Before move into different isolation levels, let's discuss different anomalies we may face when handing concurrent transactions.

## Anomalies

### Dirty Read

> A Dirty Read happens when a transaction is allowed to read uncommited data that is being modified by another transaction.

Assume we have a system which calculate monthly charge based on user balance. Let's say it's `0.1%` of current balance. Now let's say this is calculated through a scheduler which runs 8 AM 1st of each month. Now assume Alice have `10000$` in account and Bob Transfers `5000$` to Alice. And due to some rare timing, these monthly charge calcuation transaction and money transfering transaction triggereted at same time. Here is how things may happens,

```text

T1 - Money Transfer Transaction
T2 - Monthly Charge Calculation

t  | T1                               | T2
0  | Read(balance = 10000)            | -
1  | Update(balance = 10000 + 5000)   | -
2  | -                                | Read(balance = 15000) <- Allowed To Read Uncommited Amount 
3  | Transaction Fails                | -
4  | -                                | CalculateCharge(15000 * 0.001 = 15)
5  | Rollback (back to 10000)         | -
6  | -                                | Save(balance - 15 = 9985) <- After Rollback balance is 10000$ 
7  | -                                | Commit

Charge Should be 10$ but calculated as 15$ !!!

```

As we can see, due to this dirty read, the user is overly charged, which is problematic. 

### Non-Repeatable Reads

> During the transaction, **a row** is retrieved twice and the values **within the row** differ between reads, due to updations by other transactions.

Let's try to understand this with an example. Let's say we have an online store, and it has 10 pencils remaining in stock.

`User1` adds 7 pencils to the cart and proceeds to checkout.

> Checkouts take a few seconds because it has to verify the address, verify the region, calculate tax, process the card transaction, and save all shipping and payment info.

Now, let's assume there is another user (`User2`) who already started checkout of 5 pencils before `User1`,
and while `User1`'s checkout is being processed, `User2` completes the checkout.
So now we have promised `User2` that we will deliver 5 pencils, but we are still processing `User1`'s 7 pencils.

Eventually we will need 12 pencils to satisfy both customers. Anyway we only have 10 pencils remaining.

Here's how things look:

```text
T1- Transaction 1 For User 1 Checkout
T2- Transaction 2 For User 2 Checkout

t  | User1 - T1                       | User2 - T2
0  | -                                | read(pencil = 10)
1  | -                                | verifyAddress()
2  | -                                | verifyRegion()
3  | -                                | calculateRegionalTaxes()
4  | -                                | payment(pencil = 5)
5  | -                                | read(pencil = 10)
6  | -                                | validate(sold>=existing)
7  | -                                | update(pencil = pencil - 5)
8  | read(pencil = 10) <-- Allowed Only To Read Commited Data   
9  | -                                | saveSummary()
10 | verifyAddress()                  | -
11 | -                                | commit() <--[1]
12 | verifyRegion()                   | -
13 | calculateRegionalTaxes()         | -
14 | payment(pencil = 7)              | -
15 | read(pencil = 5) <--[2]          | -
16 | validate(sold>=existing) <--[3]  | -   

[1] - Now system really has 5 pencils only
[2] - In between first and second read, values have changed
[3] - Insufficient pencils, Validation fails, Have to all rollback everything, including payments 

```

Yes, things become problematic if we don't handle these scenarios properly in concurrent transactional environments.
Based on our use cases, we should take appropriate measures and choose proper isolation level.

Note : There are robust mechanisms to handle transactions on online shopping applications. This example is simplified for explanation purposes

### Phantom Reads

> Phantom read is a scenario where **two identical queries** are executed at two different times (execution 1 and execution 2) within a **single trasactional context**, yet these two queries return **different sets of rows**.

This happens when, between the execution of identical queries within a given transaction, 
some other transaction changes the data in a way that causes the conditions of these queries to satisfy more or fewer rows than the initial execution.

**Example**

Initial DB

Username    | Email                  | UserType | Amount
------------|------------------------|----------|--------
alice       | alice@example.com      | premium  | 410.50
bob         | bob@domain.com         | premium  | 340.00
carol       | carol@mail.com         | guest    | 0.00
dave        | dave@shopnow.com       | regular  | 89.99
eve         | eve@securemail.org     | premium  | 500.00


Now let's say our query, `q` is

`
q = SELECT *
FROM users utbl
WHERE utbl.UserType = 'premium' AND utbl.Amount > 400;
`

```text
t |  T1                                |  T2
0 |  execute(q)-->[alice,eve]          |  -
1 |  -                                 |  commit(bob amount += 100)--> Now bob has 440 
2 |  execute(q)-->[alice,bob,eve]      |  -
```

As we can see, now within same transaction T1, two identical queries q, but two different result sets. This case called as `Phantom Reads`.

That's all about common anomalies in DB Transactions for now. In next part we will disccuss, different isolation levels we can use to address these issues.

Hope this article was helpful.\
Thank You !\
Happy Coding 🙌