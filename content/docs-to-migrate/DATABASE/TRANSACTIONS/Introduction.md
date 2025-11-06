---
# 786
weight: 300
---

# DB Transactions : Introduction

## Definition

> A database transaction is a sequence of one or more database operations treated as a single unit of work, making all operations either complete successfully together or none at all.

## Problem

Let's begin with an example.

Let's say we want to buy an item from an online store.\
Assume it works as follows:
When we enter everything and click the "Pay and Place Order" button.

In the DB:

1. Create Order
2. Save Fund Transaction Success Info
3. Decrease Item from Store
4. Save Order Delivery Details
5. Update Order with Fund and Delivery Info

Now let's assume after step 4 the system crashed and the whole process halted.

***What will happen?***

User already paid, Item deducted, Delivery details added, but order details are not updated with those information.

Which leads to incomplete partial data of order. Which means these sequences of operations from 1–5 should happen as a single unit of work.

## Solution

To solve this type of issue is straightforward, all these operations should complete successfully together with success, or if at least some failed, all these operations should be rolled back.\
Simply put:

> Either completely happen successfully or nothing should happens at all.

These kinds of operations we call ***Single Logical Unit*** of work, which is known as a ***Transaction***.\
So we do not have to worry about partial completion of these operations.

## Lifecycle

Now let's see the lifecycle of database transactions.

![](/images/transaction1.png)

## States

During this execution, let's see how the state of a Database Transaction changes.

![](/images/transaction2.png)

- **A (Begin → Active)**: Transaction begins and enters Active State.
- **B (Active → Active)**: The transaction will be in Active mode until all the statements are executed successfully.
- **C (Active → Failed)**: If at least one statement fails, the process will halt and move to the Failure state.
- **H (Failed → Rollback)**: When failure is detected, the changes made will be ignored and not committed into the DB.
- **I (Rollback → Terminated)**: Process leaves the transaction mode.
- **D (Active → Partially Committed)**: All the executions of statements are successful but data is not committed to Disk yet (still in memory buffer).
- **G (Partially Committed → Failed)**: Due to some issues (System crash, Network issues, IO issues...), writing to Disk fails, then the state will enter the Failed state.
- **E (Partially Committed → Committed)**: All the transaction updates are successfully and permanently stored in Disk.
- **F (Committed → Terminated)**: Process leaves the transaction mode.

## How to Implement

### MySQL

```sql
START TRANSACTION;
UPDATE users SET balance = balance - 100 WHERE name = 'Alice';
UPDATE users SET balance = balance + 100 WHERE name = 'Bob';
COMMIT;
```

If something went wrong, use:

```sql
ROLLBACK;
```

Otherwise, these changes may still appear in the current session until it gets closed.

### Java

```java
conn.setAutoCommit(false); // Start transaction
try {
    PreparedStatement stmt1 = conn.prepareStatement(
        "UPDATE users SET balance = balance - 100 WHERE name = 'Alice'"
    );
    PreparedStatement stmt2 = conn.prepareStatement(
        "UPDATE users SET balance = balance + 100 WHERE name = 'Bob'"
    );
    stmt1.executeUpdate();
    stmt2.executeUpdate();
    conn.commit(); // Save changes
    System.out.println("Transaction committed.");
} catch (Exception e) {
    conn.rollback(); // Undo if something fails
    System.out.println("Transaction rolled back.");
}
```

### Spring Boot

```java
@Transactional
public void transfer(String from, String to, double amount) {
    User sender = repo.findByName(from).orElseThrow();
    User receiver = repo.findByName(to).orElseThrow();
    
    if (sender.getBalance() < amount) {
        throw new IllegalArgumentException("Insufficient funds for " + from);
    }
    
    sender.setBalance(sender.getBalance() - amount);
    receiver.setBalance(receiver.getBalance() + amount);

    repo.save(sender);
    repo.save(receiver);
}
```

Next, we will discuss about ACID Properties in Database Transactions.

Hope this article was helpful. 

Thank you!

Happy Coding 🙌

