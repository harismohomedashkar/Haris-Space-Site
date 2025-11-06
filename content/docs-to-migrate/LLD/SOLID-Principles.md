---
weight: 100
---
# SOLID Principles

![](/images/Screenshot%202025-02-11%20084203.png)

## Introduction

In this article we are going to see what are solid principles and why we need those in day to day programming.
The SOLID is short form for, ***Set of best practices which is defined to avoid code smells, duplications, refactoring, and other issues we may face while maintaining and extending as project evolves and grows***.

SOLID stands for:

- ***S*** \- Single\-responsibility Principle
- ***O*** \- Open\-closed Principle
- ***L*** \- Liskov Substitution Principle
- ***I*** \- Interface Segregation Principle
- ***D*** \- Dependency Inversion Principle

Let's see each of this one by one with real\-world examples.

## Single\-Responsibility Principle

The Single Responsibility Principle tells us, 

> We should plan and create classes and its hierarchy in a way that each class have it's specific scope where it is responsible for.

Let's consider following example,

Let's say we have a *UserManager* Class which has methods for *save, findById, isUserActive, isValidUser, sendWelcomeEmail* etc. As we can see this class handles multiple responsibilities itself, alone. This is an example of ***violating*** single responsible principle.

The correct way is we should define separate classes to handle different scopes. Following is a way we can do this,

* Database Operations: Methods like save\(\) and findById\(\) should be in UserRepository class.
* Business Logic: isUserActive\(\) should be in a UserService class.
* Validation Logic: isValidUser\(\) should be in a UserValidator class.
* External Service Calls: sendWelcomeEmail\(\) should be handled by a separate EmailService.

This way we can comply with *Single Responsibility Principle*.

## Open Closed Principle

This means our program 

> Classes should be closed for modification, but open for extension. 

Let's look at this with an example.
Let's say we have *UserRepository* and it has methods to read/write values from/into DB. And these methods *should support both MySQL and PostgreSQL*. Consider following example.

```java
import java.util.List;

interface Database {
    List<Object> readAll();
}

class MySQL implements Database {
    public List<Object> readAll() {
        return List.of("Some", "Data", "From", "MySQL", "DB");
    }
}

class PostgreSQL implements Database {
    public List<Object> readAll() {
        return List.of("Some", "Data", "From", "PostgreSQL", "DB");
    }
}

class UserRepository {
    private Database db;

    public UserRepository(String dbType) {
        switch (dbType) {
            case "MySQL":
                db = new MySQL();
                break;
            case "PostgreSQL":
                db = new PostgreSQL();
                break;
            default:
                throw new RuntimeException("Unsupported Database Type: " + dbType);
        }
    }

    public List<Object> readRows() {
        return db.readAll();
    }
}
```
```java
UserRepository mySQL = new UserRepository("MySQL");
System.out.println(mySQL.readRows());

UserRepository postgreSQL = new UserRepository("PostgreSQL");
System.out.println(postgreSQL.readRows());
```

Now, let's say we need to add support for one more DB Type MSSQL. How we achieve this is

1. Create a class for new DB implements already existing Database interface.
2. Add new case statement in UserRepository

Updated code as follows,

```java
import java.util.List;

interface Database {
    List<Object> readAll();
}

class MySQL implements Database {
    public List<Object> readAll() {
        return List.of("Some", "Data", "From", "MySQL", "DB");
    }
}

class PostgreSQL implements Database {
    public List<Object> readAll() {
        return List.of("Some", "Data", "From", "PostgreSQL", "DB");
    }
}

// ADD NEW CLASS
class MSSQL implements Database {
    public List<Object> readAll() {
        return List.of("Some", "Data", "From", "MSSQL", "DB");
    }
}

class UserRepository {
    private Database db;

    public UserRepository(String dbType) {
        switch (dbType) {
            case "MySQL":
                db = new MySQL();
                break;
            case "PostgreSQL":
                db = new PostgreSQL();
                break;
            // ADD NEW CASE STATEMENT
            case "MSSQL":
                db = new MSSQL();
                break;
            default:
                throw new RuntimeException("Unsupported Database Type: " + dbType);
        }
    }

    public List<Object> readRows() {
        return db.readAll();
    }
}
```
```java
UserRepository mySQL = new UserRepository("MySQL");
System.out.println(mySQL.readRows());

UserRepository postgreSQL = new UserRepository("PostgreSQL");
System.out.println(postgreSQL.readRows());

UserRepository msSQL = new UserRepository("MSSQL");
System.out.println(msSQL.readRows());
```

Technically code works fine with no issues. But when we look longer term, everything we update support or logic for a certain DB, we have to touch *UserRepository* Class. This should be avoided according to *Open Closed Principal* in SOLID. ***Open Closed Principle says classes should be Open to extension but should be closed for modification***.

Now let's look Code Below,
```java
import java.util.List;

interface Database {
    List<Object> readAll();
}

class MySQL implements Database {
    public List<Object> readAll() {
        return List.of("Some", "Data", "From", "MySQL", "DB");
    }
}

class PostgreSQL implements Database {
    public List<Object> readAll() {
        return List.of("Some", "Data", "From", "PostgreSQL", "DB");
    }
}

interface DatabaseFactory {
    Database createDatabase();
}

class MySQLFactory implements DatabaseFactory {
    public Database createDatabase() {
        return new MySQL();
    }
}

class PostgreSQLFactory implements DatabaseFactory {
    public Database createDatabase() {
        return new PostgreSQL();
    }
}

class UserRepository {
    private Database db;

    public UserRepository(DatabaseFactory factory) {
        this.db = factory.createDatabase();
    }

    public List<Object> readRows() {
        return db.readAll();
    }
}
```
```java
UserRepository mysqlRepo = new UserRepository(new MySQLFactory());
System.out.println("MySQL Data: " + mysqlRepo.readRows());

UserRepository postgresRepo = new UserRepository(new PostgreSQLFactory());
System.out.println("PostgreSQL Data: " + postgresRepo.readRows());
```

Now let's think of adding new MSSQL support. This time we don't need to touch any exisiting class. Just need to add new classes to with relvant logic into program.

1. Create MSSQL Database Class.
2. Create MSSQL DatabaseFactory Class.

Let's look into following code.

```java
import java.util.List;

interface Database {
    List<Object> readAll();
}

class MySQL implements Database {
    public List<Object> readAll() {
        return List.of("Some", "Data", "From", "MySQL", "DB");
    }
}

class PostgreSQL implements Database {
    public List<Object> readAll() {
        return List.of("Some", "Data", "From", "PostgreSQL", "DB");
    }
}

class MSSQL implements Database {
    public List<Object> readAll() {
        return List.of("Some", "Data", "From", "MSSQL", "DB");
    }
}

interface DatabaseFactory {
    Database createDatabase();
}

class MySQLFactory implements DatabaseFactory {
    public Database createDatabase() {
        return new MySQL();
    }
}

class PostgreSQLFactory implements DatabaseFactory {
    public Database createDatabase() {
        return new PostgreSQL();
    }
}

class MSSQLFactory implements DatabaseFactory {
    public Database createDatabase() {
        return new MSSQL();
    }
}

class UserRepository {
    private Database db;

    public UserRepository(DatabaseFactory factory) {
        this.db = factory.createDatabase();
    }

    public List<Object> readRows() {
        return db.readAll();
    }
}
```
```java
UserRepository mysqlRepo = new UserRepository(new MySQLFactory());
System.out.println("MySQL Data: " + mysqlRepo.readRows());

UserRepository postgresRepo = new UserRepository(new PostgreSQLFactory());
System.out.println("PostgreSQL Data: " + postgresRepo.readRows());

UserRepository msRepo = new UserRepository(new MSSQLFactory());
System.out.println("MSSQL Data: " + msRepo.readRows());
```

This shows why we really need *Open\-Closed Principle* for better programming.

## Liskov Substitution Principle

The Liskov Substitution Principle \(LSP\) states:

> Objects of a superclass should be replaceable with objects of a subclass without altering the correctness of the program.

 
Let's look into this with above example,

So the, Super type is **DatabaseFactory**, Subtypes are **MySQLFactory, PostgreSQLFactory, MSSQLFactory**

According to *Liskov Substitution Principle* we should be able to use any of subtypes, where *DatabaseFactory* is the type required. So the **UserRepsotory should work as same way regardless of DatabaseFactory is MySQLFactory, PostgreSQLFactory, MSSQLFactory.**

Now let's say we have a **InMemoryDatabaseFactory** class, which does not support createDatabase method as shown below. This will ***violate*** Liskov Substitution Principle, because we can't expect same behaviour when we replaced another DatabaseFactory with this.

```java
class InMemoryDatabaseFactory implements DatabaseFactory {
    // Violates LSP because it does not implement `createDatabase`
    public Database createDatabase() {
        throw new UnsupportedOperationException("Not Suppported");
    }
}
```

## Interface Segregation Principle

The Interface Segregation Principle \(ISP\) states:

> A client should not be forced to depend on methods it does not use.


Let's look this with our example. Let's say we have new type of database, *ReadOnlyMySQL* which is used where read\-only access should be given, for security purposes. So the code will be as follows,

```java
interface Database {
    List<Object> readAll();
    void insert(Object obj);
    void update(Object obj);
    void delete(Object obj);
}

class MySQL implements Database {
    public List<Object> readAll() { return List.of("Some", "Data", "From", "MySQL", "DB"); }
    public void insert(Object obj) { System.out.println("Interting Data . . ."); }
    public void update(Object obj) { System.out.println("Updating Data . . ."); }
    public void delete(Object obj) { System.out.println("Deleting Data . . ."); }
}

class ReadOnlyMySQL implements Database {
    public List<Object> readAll() { return List.of("Read-Only", "Data", "From", "Database"); }
    public void insert(Object obj) { throw new UnsupportedOperationException(); }
    public void update(Object obj) { throw new UnsupportedOperationException(); }
    public void delete(Object obj) { throw new UnsupportedOperationException(); }
}
```

As we can see we are ***forcefully adding unsupported*** insert, update, delete methods into our *ReadOnlyMySQL* class. This ***violates*** Interface Segregation principle. So now let's see how we can fix this.

```java
interface ReadableDatabase {
    List<Object> readAll();
}

interface WritableDatabase {
    void insert(Object obj);
    void update(Object obj);
    void delete(Object obj);
}
```
First we created two interfaces to seperate read and write methods.

```java
class MySQL implements ReadableDatabase, WritableDatabase{
    public List<Object> readAll() { return List.of("Some", "Data", "From", "MySQL", "DB"); }
    public void insert(Object obj) { System.out.println("Interting Data . . ."); }
    public void update(Object obj) { System.out.println("Updating Data . . ."); }
    public void delete(Object obj) { System.out.println("Deleting Data . . ."); }
}

class ReadOnlyMySQL implements ReadableDatabase {
    public List<Object> readAll() { return List.of("Read-Only", "Data", "From", "Database"); }
}
```
As we can see now, classes only implement what it needs. So this complies to interface segregation principal.

## Dependency Inversion Principle

The Dependency Inversion Principle \(DIP\) states:

> High\-level modules should not depend on low\-level modules. Both should depend on abstractions.

> Abstractions should not depend on details. Details should depend on abstractions.

Now let's move back to our old code again

```java
class UserRepository {
    private Database db;

    public UserRepository(String dbType) { 
        switch (dbType) {
            case "MySQL":
                db = new MySQL();
                break;
            case "PostgreSQL":
                db = new PostgreSQL();
                break;
            default:
                throw new RuntimeException("Unsupported Database Type: " + dbType);
        }
    }

    public List<Object> readRows() {
        return db.readAll();
    }
}
```
Let's see why this violates Dependency Inversion Principle,

1. High\-level module \(UserRepository\) depends on low\-level classes \(MySQL, PostgreSQL\).
2. This is because tighly couple between UserRepository and Databases. Because objects of Databases are created within the UserRepository and In case we need to add support for new Database we have to change UserRepository code.
3. When it comes to unit test, we can't replace Database in UserRepository with a mock class.

Let's fix this.
```java
class DatabaseFactory {
    public static Database getDatabase(String dbType) {
        switch (dbType) {
            case "MySQL":
                return new MySQL();
            case "PostgreSQL":
                return new PostgreSQL();
            default:
                throw new RuntimeException("Unsupported Database Type: " + dbType);
        }
    }
}
```
We moved the creation logic of database to seperate Factory Class.
```java
class UserRepository {
    private Database db;

    public UserRepository(Database db) {
        this.db = db;
    }

    public List<Object> readRows() {
        return db.readAll();
    }
}
```
Now Inject the database classes created by that factory into UserRepository at runtime in the constructor. Now,

1. UserRepository depends on an abstraction \(Database\), not on concrete classes \(MySQL, PostgreSQL\)
2. Adding support to new Database requires no changes in UserRepository. UserRepository does not care which database is injected. it just works with Database
3. We can inject a mock database in unit tests


That's all with SOLID principal for now.

Happy Coding 🙌
