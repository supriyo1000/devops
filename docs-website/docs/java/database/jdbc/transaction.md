# Phase 7 — 7.5 Database Transactions: Isolation Levels

Now let's move to the next topic: **Transaction Isolation and Concurrency Problems**.

This is an important Java/Spring Boot interview topic because real backend applications often have multiple users accessing and modifying the same database records simultaneously.

## What We'll Learn

1. Why transaction isolation is needed.
2. Dirty Read.
3. Non-Repeatable Read.
4. Phantom Read.
5. READ COMMITTED.
6. REPEATABLE READ.
7. SERIALIZABLE.
8. How to configure isolation in JDBC.
9. How to configure isolation in Spring Boot.
10. Practical interview scenarios.

---

## 1. Why Do We Need Transaction Isolation?

Imagine your application has a bank account:

- Account ID: `101`
- Balance: ₹10,000

Two users access this account simultaneously.

- Transaction T1 reads the balance.
- Transaction T2 reads the balance.
- T1 updates the balance.
- T2 also updates the balance.

Now both transactions are interacting with the same database record.

This is called **concurrent transaction execution**.

Without appropriate concurrency control, transactions can interfere with each other.

### Transaction Isolation

Isolation determines how much one transaction can see the changes made by other concurrently executing transactions.

Think of it this way:

```text
              PostgreSQL
                  |
          ┌───────┴───────┐
          |               |
     Transaction T1  Transaction T2
          |               |
          └───────┬───────┘
                  |
           Isolation Rules
                  |
          Consistent behavior
```

The SQL standard defines four isolation levels:

| Isolation Level | Strength |
|---|---|
| READ UNCOMMITTED | Lowest |
| READ COMMITTED | Low |
| REPEATABLE READ | Higher |
| SERIALIZABLE | Highest |

Higher isolation generally provides stronger guarantees but can reduce concurrency or cause transactions to wait or retry.

---

## 2. Dirty Read

**Important interview concept**

A dirty read occurs when one transaction reads data modified by another transaction that has not yet committed.

### Example

Initial balance:

```text
Account A = ₹10,000
```

Two transactions:

### Transaction T1

```sql
BEGIN;

UPDATE accounts
SET balance = 5000
WHERE id = 1;
```

T1 has not committed yet.

### Transaction T2

```sql
BEGIN;

SELECT balance
FROM accounts
WHERE id = 1;
```

Suppose T2 reads:

```text
₹5,000
```

But then T1 executes:

```sql
ROLLBACK;
```

The actual balance returns to:

```text
₹10,000
```

T2 has read a value that was never committed.

That is a **dirty read**.

### Dirty Read — Timeline

1. T1 updates balance:

   ```text
   ₹10,000 → ₹5,000 (uncommitted)
   ```

2. T2 reads balance:

   ```text
   Reads ₹5,000 (dirty value)
   ```

3. T1 rolls back:

   ```text
   Balance becomes ₹10,000 again
   ```

4. T2 has incorrect information:

   It previously read a value that was rolled back.

### Interview Question: Can Dirty Reads Happen in PostgreSQL?

No. PostgreSQL does not allow dirty reads, even when you set `READ UNCOMMITTED`. It behaves as `READ COMMITTED`.

---

## 3. Non-Repeatable Read

A non-repeatable read occurs when the same transaction reads the same row twice but gets different values because another transaction committed an update between the reads.

### Example

Initial balance:

```text
Account A = ₹10,000
```

### Transaction T1

```sql
BEGIN;

SELECT balance
FROM accounts
WHERE id = 1;
```

Result:

```text
₹10,000
```

Now T2 executes:

```sql
BEGIN;

UPDATE accounts
SET balance = 8000
WHERE id = 1;

COMMIT;
```

T1 executes the same query again:

```sql
SELECT balance
FROM accounts
WHERE id = 1;
```

Result:

```text
₹8,000
```

T1 got two different results for the same row.

That is a **non-repeatable read**.

### Non-Repeatable Read

```text
T1 — First read
₹10,000

T1 — Second read
₹8,000
```

T2 committed an update between T1's two reads.

**Key point:** The row still exists. Its value changed.

---

## 4. Phantom Read

A phantom read occurs when the same transaction executes the same search condition twice but gets a different set of rows because another transaction inserted or deleted matching records.

### Example

Suppose an `employees` table contains:

| ID | Name | Salary |
|---|---|---:|
| 1 | Rahul | 30,000 |
| 2 | Amit | 40,000 |

T1 executes:

```sql
BEGIN;

SELECT *
FROM employees
WHERE salary > 25000;
```

Result: 2 employees.

Now T2 executes:

```sql
BEGIN;

INSERT INTO employees(name, salary)
VALUES ('Priya', 35000);

COMMIT;
```

T1 executes the same query again:

```sql
SELECT *
FROM employees
WHERE salary > 25000;
```

Now it gets 3 employees.

The newly appearing row is called a **phantom row**.

### Difference Between Non-Repeatable Read and Phantom Read

| Non-Repeatable Read | Phantom Read |
|---|---|
| Existing row changes | Matching row set changes |
| UPDATE commonly causes it | INSERT/DELETE commonly causes it |
| Same row, different value | Different result set |

---

## 5. READ COMMITTED

<badge>PostgreSQL default</badge>

`READ COMMITTED` guarantees that a transaction only reads committed data.

### It Prevents

- Dirty reads.

### It Does Not Prevent

- Non-repeatable reads.
- Phantom reads.

### Example

```sql
BEGIN TRANSACTION
ISOLATION LEVEL READ COMMITTED;

SELECT balance
FROM accounts
WHERE id = 1;

-- Another transaction updates and commits the balance.

SELECT balance
FROM accounts
WHERE id = 1;

COMMIT;
```

The two `SELECT` statements can return different values.

### Why?

Because PostgreSQL creates a new snapshot for each statement under `READ COMMITTED`.

### Remember

| Problem | READ COMMITTED |
|---|---|
| Dirty Read | Prevented |
| Non-Repeatable Read | Possible |
| Phantom Read | Possible |

---

## 6. REPEATABLE READ

`REPEATABLE READ` ensures that repeated reads within the same transaction see a consistent snapshot.

### It Prevents

- Dirty reads.
- Non-repeatable reads.
- Phantom reads in PostgreSQL.

### Example

```sql
BEGIN TRANSACTION
ISOLATION LEVEL REPEATABLE READ;

SELECT balance
FROM accounts
WHERE id = 1;

-- Another transaction updates and commits.

SELECT balance
FROM accounts
WHERE id = 1;

COMMIT;
```

Both `SELECT` statements see the same transaction snapshot.

Even if another transaction updates the record and commits, T1 continues seeing the older committed version.

This is possible because PostgreSQL uses **MVCC (Multi-Version Concurrency Control)**.

### What Is MVCC?

MVCC allows PostgreSQL to maintain multiple versions of a row.

For example:

```text
Account ID: 1

Version 1: ₹10,000
Version 2: ₹8,000
Version 3: ₹5,000
```

A transaction reads the appropriate row version according to its snapshot.

This reduces the need for readers and writers to block each other.

### Remember

| Problem | REPEATABLE READ (PostgreSQL) |
|---|---|
| Dirty Read | Prevented |
| Non-Repeatable Read | Prevented |
| Phantom Read | Prevented |

**Important advanced point:** PostgreSQL's `REPEATABLE READ` is stronger than the minimum SQL-standard requirement for phantom reads, but it can still allow serialization anomalies.

---

## 7. SERIALIZABLE

<badge>Highest isolation level</badge>

`SERIALIZABLE` provides the strongest transaction isolation.

It ensures that successfully committed concurrent transactions produce an outcome equivalent to some serial execution order.

Imagine:

```text
Transaction T1

Transaction T2
```

Instead of allowing unsafe concurrent behavior, the database detects certain conflicting execution patterns.

It may abort one transaction with a serialization failure.

### Example

Suppose a hospital has two doctors on duty.

**Rule:** At least one doctor must remain on duty.

Initial state:

| Doctor | On Duty |
|---|---|
| Doctor A | Yes |
| Doctor B | Yes |

T1 checks that Doctor B is on duty, then decides Doctor A can go off duty.

T2 checks that Doctor A is on duty, then decides Doctor B can go off duty.

Both transactions independently see one doctor remaining on duty.

Under insufficient isolation, both might commit:

| Doctor | On Duty |
|---|---|
| Doctor A | No |
| Doctor B | No |

This violates the business rule.

`SERIALIZABLE` detects this kind of dangerous concurrency pattern and can abort one transaction.

### SQL Syntax

```sql
BEGIN TRANSACTION
ISOLATION LEVEL SERIALIZABLE;

SELECT *
FROM employees;

-- Perform business operations.

COMMIT;
```

If PostgreSQL detects a serialization conflict, it can return:

```text
SQLSTATE: 40001
serialization_failure
```

Your application should handle this by retrying the entire transaction when appropriate.

### Remember

| Problem | SERIALIZABLE |
|---|---|
| Dirty Read | Prevented |
| Non-Repeatable Read | Prevented |
| Phantom Read | Prevented |
| Serialization Anomaly | Detected/prevented |

---

## 8. Isolation Levels — Complete Revision Table

This table is particularly important for interviews.

### SQL Standard Behavior

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|---|---|---|---|
| READ UNCOMMITTED | Possible* | Possible | Possible |
| READ COMMITTED | No | Possible | Possible |
| REPEATABLE READ | No | No | Possible* |
| SERIALIZABLE | No | No | No |

\* This is the general SQL-standard behavior. PostgreSQL prevents dirty reads at every level and prevents phantom reads under `REPEATABLE READ`.

### PostgreSQL-Specific Behavior

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|---|---|---|---|
| READ UNCOMMITTED | No | Possible | Possible |
| READ COMMITTED | No | Possible | Possible |
| REPEATABLE READ | No | No | No |
| SERIALIZABLE | No | No | No |

---

## 9. JDBC — Set Transaction Isolation Level

JDBC provides a method:

```java
connection.setTransactionIsolation();
```

It accepts these constants:

```java
Connection.TRANSACTION_READ_UNCOMMITTED

Connection.TRANSACTION_READ_COMMITTED

Connection.TRANSACTION_REPEATABLE_READ

Connection.TRANSACTION_SERIALIZABLE
```

### Example

```java
try (Connection connection =
         dataSource.getConnection()) {

    connection.setAutoCommit(false);

    connection.setTransactionIsolation(
        Connection.TRANSACTION_REPEATABLE_READ
    );

    try {

        // Execute SQL operations

        connection.commit();

    } catch (SQLException e) {

        connection.rollback();

        throw e;
    }
}
```

### Check the Current Isolation Level

```java
int isolationLevel =
    connection.getTransactionIsolation();

System.out.println(isolationLevel);
```

**Note:** Configure the isolation level before executing transaction statements. Database-specific support and behavior can vary.

---

## 10. Spring Boot — @Transactional Isolation

In Spring Boot, you can configure isolation directly using `@Transactional`.

### READ COMMITTED

```java
@Transactional(
    isolation = Isolation.READ_COMMITTED
)
public void updateEmployee() {

    // Database operations
}
```

### REPEATABLE READ

```java
@Transactional(
    isolation = Isolation.REPEATABLE_READ
)
public void processEmployee() {

    // Database operations
}
```

### SERIALIZABLE

```java
@Transactional(
    isolation = Isolation.SERIALIZABLE
)
public void processPayment() {

    // Database operations
}
```

### Available Spring Isolation Values

```java
Isolation.DEFAULT

Isolation.READ_UNCOMMITTED

Isolation.READ_COMMITTED

Isolation.REPEATABLE_READ

Isolation.SERIALIZABLE
```

`Isolation.DEFAULT` means Spring uses the database's default isolation level.

For PostgreSQL, this is `READ COMMITTED`.

### Important Interview Point

`@Transactional` normally applies through Spring's transaction proxy.

Calling another transactional method through `this.method()` within the same object does not normally trigger a new proxy interception.

Also, isolation settings generally matter when a new transaction is started; joining an existing transaction does not automatically change its isolation level.

---

## 11. Practical Experiment — Test Isolation in PostgreSQL

This is the most useful exercise to understand isolation rather than simply memorizing definitions.

Open two PostgreSQL terminals:

- Terminal 1 = Transaction T1
- Terminal 2 = Transaction T2

### Create Test Table

```sql
CREATE TABLE accounts (
    id SERIAL PRIMARY KEY,
    balance NUMERIC(12, 2)
);

INSERT INTO accounts(balance)
VALUES (10000);
```

### Experiment A — READ COMMITTED

**Terminal 1**

```sql
BEGIN TRANSACTION
ISOLATION LEVEL READ COMMITTED;

SELECT balance
FROM accounts
WHERE id = 1;
```

Expected:

```text
10000
```

**Terminal 2**

```sql
BEGIN;

UPDATE accounts
SET balance = 8000
WHERE id = 1;

COMMIT;
```

**Terminal 1**

```sql
SELECT balance
FROM accounts
WHERE id = 1;
```

Expected:

```text
8000
```

**Why?** Each statement gets a fresh snapshot.

### Experiment B — REPEATABLE READ

Reset the balance:

```sql
UPDATE accounts
SET balance = 10000
WHERE id = 1;
```

**Terminal 1**

```sql
BEGIN TRANSACTION
ISOLATION LEVEL REPEATABLE READ;

SELECT balance
FROM accounts
WHERE id = 1;
```

Expected:

```text
10000
```

**Terminal 2**

```sql
BEGIN;

UPDATE accounts
SET balance = 8000
WHERE id = 1;

COMMIT;
```

**Terminal 1**

```sql
SELECT balance
FROM accounts
WHERE id = 1;
```

Expected:

```text
10000
```

T1 still sees its original snapshot.

Finally:

```sql
COMMIT;
```