---
# 786
weight: 400
---

# DB Transactions : ACID

## Rule of Thumb

> ALL DB OPERATIONS, INCLUDING THE EXECUTION OF A SINGLE STATEMENT, ARE ALSO TRANSACTIONS.

## Introduction

In [Introduction](/docs/database/transactions/introduction/) we looked at what transactions mean, why we need them, and how they work. In this part, we will discuss the properties that transactions should have. There are four well known properties that transactions should comply with.

These properties are:

- **A**tomicity
- **C**onsistency
- **I**solation
- **D**urability

We call these properties the **ACID** Properties in databases. Now let's discuss each one, one by one.

## Properties

### Atomicity

Atomicity makes sure that all operations within the transaction are handled as a **single unit of work**. That means either all operations are completed or nothing is completed at all.
The purpose of this property is to avoid partially completed states, which lead to inconsistencies in the database.

For example, let's take a popular example: 

Bank transfer. If A transfers $2000 to B’s account:

- The amount should be deducted from A’s account
- The amount should be added to B’s account

These operations should happen as a single unit of work.\
Otherwise, if the system crashes after money is deducted from A but before it's added to B,\
there will be a state where money left A but never reached B, which will be problematic.

- If all operations within the transaction are completed successfully → **Commit** (Permanently written to disk)
- If at least one operation within the transaction fails → **Rollback** (Every change made within the transaction will be ignored)


### Consistency

When it comes to databases, they have several schemas, tables, and data. Based on our business requirements, we define constraints and rules on top of these.

When all these -> data type checks, null checks, foreign keys, relationships, unique keys, etc. are followed by the data, we call it a **valid state of the database**.

So, the Consistency in ACID means that a ***transaction should move the DB state from one valid state to another valid state***.
There should not be any invalid state the database enters due to a transaction. ***If the DB state becomes invalid, the transaction should be rolled backed***.

This ensures the transaction will not corrupt DB data or constraints.

### Isolation

Databases can handle multiple concurrent transactions at once. The Isolation property make sure that ***multiple transactions can occur concurrently*** without leading the database state into inconsistency eventually.

According to this property, it is ensured that once started, a transaction ***has limitations*** on what it ***can see of changes*** made by ***other transactions***.

This limitation is called the ***Isolation Level of the transaction***, which defines limit of what the transaction can see of what other transactions are changing.

We can change this isolation level based on our business requirements.

However, different databases have different default isolation levels (we can change them to specific values as needed).

For example, `MySQL (InnoDB)` has a default level called `Repeatable Read`,
which means once a transaction starts, throughout the transaction, it only sees:

```text
Data read at the initial state 
    +
Data changed by itself
```

It can’t see any changes (committed or uncommitted) made by other transactions at all.


### Durability

This means once a transaction is committed, the changes made by the statements in the transaction will be written to
***persistent storage*** to make sure the changes are permanent and ***will survive any failures, crashes, power outages, etc. afterward***.

This ensures that the data / db state will not be corrupted or lost.

## Shared Responsibility

Now let's look at whose responsibility it is to maintain these properties:

1. Atomicity:
    
    Developers should identify which statements should be executed as a single unit and mark them as a transaction. Once marked as transaction, it is the database's responsibility to handle them.

2. Consistency: 

    Developers should define null/non-null constraints, data types, unique keys, foreign keys, and other rules while creating the database structure. Once constraints and rules are defined, the DB will handle it.

3.  Isolation: 

    Developers should assume that a transaction may be interrupted by other transactions and should define the isolation level a transaction should follow. Once defined, it is the DB's responsibility to handle it.

4. Durability: 

    This is handled by the database engine and the underlying storage system.


Yeah, that’s all about ACID in database transactions for now.

Hope this article was helpful.

Thank you !

Happy Coding 🙌