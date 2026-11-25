# Phase 7.6 — Database Concurrency Problems & Locking

Now we move from understanding transaction isolation to solving real backend problems.

In real applications, multiple requests can access the same data simultaneously. Even if every request is correct individually, concurrent execution can create incorrect results.

This is especially important in Spring Boot applications involving:

- Banking and payments
- Inventory management
- Order processing
- Booking systems
- Employee management
- Distributed APIs

## Topics We Will Cover

- 7.6 Database Concurrency
- 7.6.1 Lost Update
- 7.6.2 Race Condition
- 7.6.3 Optimistic Locking
- 7.6.4 Pessimistic Locking
- 7.6.5 Database Locks
- 7.6.6 Deadlock
- 7.6.7 Retry Strategy
- 7.6.8 Spring Boot Implementation

---

## 7.6.1 Lost Update

### What Is a Lost Update?

A lost update occurs when two transactions update the same record concurrently and one update overwrites the other, causing one change to be lost.

### Example: Inventory

Your database:

```sql
CREATE TABLE products (
    id INT PRIMARY KEY,
    product_name VARCHAR(100),
    stock INT
);

INSERT INTO products
VALUES (1, 'Laptop', 10);
```

Initial stock:

```text
10 laptops
```

### Available Inventory

Two users purchase one laptop each at nearly the same time.

### Transaction T1

```sql
BEGIN;

SELECT stock
FROM products
WHERE id = 1;
```

T1 reads:

```text
10
```

### Transaction T2

```sql
BEGIN;

SELECT stock
FROM products
WHERE id = 1;
```

T2 also reads:

```text
10
```

Both transactions independently calculate:

```text
10 - 1 = 9
```

### T1

```sql
UPDATE products
SET stock = 9
WHERE id = 1;

COMMIT;
```

### T2

```sql
UPDATE products
SET stock = 9
WHERE id = 1;

COMMIT;
```

Final database state:

```text
Stock = 9
```

But two laptops were sold.

The correct result should have been:

```text
Stock = 8
```

### Lost Update

Two updates occurred, but one logical change was lost because both transactions calculated the result from the same old value.

### Why Did This Happen?

Both transactions followed the same process:

```text
READ STOCK
     |
     ▼
CALCULATE NEW STOCK IN JAVA
     |
     ▼
UPDATE STOCK
```

The problem is that the calculation happened using stale data.

### How to Prevent Lost Updates

There are several approaches:

| Approach | Mechanism |
|---|---|
| Atomic SQL update | Database performs calculation |
| Pessimistic locking | Lock the row while working |
| Optimistic locking | Detect whether data changed |
| Serializable isolation | Detect conflicting execution patterns |

We'll learn each approach.

---

## 7.6.2 Solution 1 — Atomic SQL Updates

The simplest solution for many inventory and counter operations is to perform the calculation inside the SQL statement rather than reading the value into Java first.

### Unsafe Approach

```java
int stock = repository.getStock(productId);

int updatedStock = stock - 1;

repository.updateStock(productId, updatedStock);
```

Why is this unsafe?

Two requests may read the same value before either updates it.

### Safer Approach

```sql
UPDATE products
SET stock = stock - 1
WHERE id = 1
  AND stock > 0;
```

The database performs the subtraction as a single atomic operation.

The `stock > 0` condition prevents the stock from becoming negative.

### In Spring Boot JdbcTemplate

```java
public boolean purchaseProduct(int productId) {

    String sql = """
        UPDATE products
        SET stock = stock - 1
        WHERE id = ?
          AND stock > 0
        """;

    int rowsAffected = jdbcTemplate.update(
        sql,
        productId
    );

    return rowsAffected == 1;
}
```

### How It Works

If stock is 10:

```text
Request 1 → stock - 1
Request 2 → stock - 1
```

PostgreSQL serializes conflicting row updates appropriately. Each statement applies against the row's current state, so the resulting stock is 8 after both successful purchases.

### Remember for Interviews

For simple counters, inventory decrements, and balance adjustments, prefer atomic SQL operations over:

```text
SELECT → calculate in Java → UPDATE
```

This is often the simplest and most effective way to prevent lost updates for a single-row operation.

---

## 7.6.3 Race Condition

A race condition occurs when the correctness of a program depends on the timing or order of concurrent operations.

Lost update is one type of race condition.

### Example: Booking Seats

```text
Available seats = 1
```

Two users try to book the last seat simultaneously.

```text
User A ────┐
           ├──► Available seats = 1
User B ────┘
```

Both requests:

1. Check whether a seat is available.
2. See 1 seat.
3. Decide booking is allowed.
4. Insert a booking.

Result:

```text
Bookings = 2
Available seats = 1
```

This is a business-rule violation.

### Unsafe Code

```java
if (bookingRepository.getAvailableSeats(eventId) > 0) {

    bookingRepository.createBooking(
        eventId,
        userId
    );
}
```

The problem is that checking and booking are two separate operations.

Another transaction can modify the data between the check and the action.

### Important Interview Distinction

- **Race condition:** General concurrency timing problem.
- **Lost update:** One particular concurrency anomaly where an update is overwritten.
- **Dirty read:** Reading uncommitted data.
- **Deadlock:** Transactions wait for each other's locks indefinitely until the database detects and aborts one.

---

## 7.6.4 Optimistic Locking

<badge>Very common in backend interviews</badge>

Optimistic locking assumes that conflicts are relatively uncommon.

Instead of locking a row every time someone reads it, we allow transactions to work normally and check whether the data changed before accepting the update.

If the data changed unexpectedly, the update is rejected.

### Concept: Version Column

Add a version column.

```sql
CREATE TABLE products (
    id BIGINT PRIMARY KEY,
    product_name VARCHAR(100),
    stock INT,
    version INT NOT NULL DEFAULT 0
);
```

Initial data:

| ID | Product | Stock | Version |
|---|---|---:|---:|
| 1 | Laptop | 10 | 0 |

Every successful update increments the version.

### How Optimistic Locking Works

Suppose two transactions read:

```text
Stock = 10
Version = 0
```

**Transaction T1**

```text
Read stock = 10
Read version = 0
```

**Transaction T2**

```text
Read stock = 10
Read version = 0
```

T1 executes:

```sql
UPDATE products
SET stock = 9,
    version = 1
WHERE id = 1
  AND version = 0;
```

Affected rows:

```text
1
```

T1 succeeds.

Now T2 executes:

```sql
UPDATE products
SET stock = 9,
    version = 1
WHERE id = 1
  AND version = 0;
```

Affected rows:

```text
0
```

Why?

The actual database version is now 1, not 0.

T2 knows that the data changed after it read the record.

### Optimistic Locking

Update only if the version is still the version I originally read.

If affected rows = 0 → conflict detected.

### SQL Implementation

```sql
UPDATE products
SET stock = ?,
    version = version + 1
WHERE id = ?
  AND version = ?;
```

The last parameter is the version previously read.

### Java Implementation Using JDBC

```java
public boolean updateStock(
        int productId,
        int newStock,
        int oldVersion
) {

    String sql = """
        UPDATE products
        SET stock = ?,
            version = version + 1
        WHERE id = ?
          AND version = ?
        """;

    int rows = jdbcTemplate.update(
        sql,
        newStock,
        productId,
        oldVersion
    );

    return rows == 1;
}
```

Usage:

```java
boolean updated = repository.updateStock(
    productId,
    9,
    0
);

if (!updated) {
    throw new IllegalStateException(
        "Product was modified by another transaction"
    );
}
```

### Optimistic Locking Workflow

```text
Read Record
    |
    ▼
Read Version = 5
    |
    ▼
Perform Business Logic
    |
    ▼
UPDATE WHERE version = 5
    |
    ├── 1 row updated → SUCCESS
    |
    └── 0 rows updated → CONFLICT
```

### When to Use Optimistic Locking?

- User profile editing
- Product details updates
- Configuration changes
- CRUD applications
- Concurrent modifications that are relatively uncommon
- Applications where blocking users is undesirable

### Advantages

- No long-held row locks during business processing.
- Better concurrency.
- Conflicts are detected instead of silently overwriting data.

### Disadvantages

- Update conflicts require handling.
- User may need to reload or retry.
- More complex business logic may need careful retry strategies.

---

## 7.6.5 Pessimistic Locking

Pessimistic locking assumes that conflicts may occur and prevents other transactions from modifying a resource while you are working with it.

It locks the row before the business operation proceeds.

### Example

Stock:

```text
10 laptops
```

T1 wants to purchase one laptop.

**T1:**

```sql
BEGIN;

SELECT stock
FROM products
WHERE id = 1
FOR UPDATE;
```

`FOR UPDATE` locks the selected row.

Now T2 attempts:

```sql
BEGIN;

SELECT stock
FROM products
WHERE id = 1
FOR UPDATE;
```

T2 must wait until T1 releases the lock.

### Timeline

**Pessimistic Lock**

```text
T1

Acquire row lock → Read stock → Decrease stock → COMMIT

T2

Attempts row lock → Waits → Reads updated stock → Continues
```

### SQL Example

```sql
BEGIN;

SELECT stock
FROM products
WHERE id = 1
FOR UPDATE;

UPDATE products
SET stock = stock - 1
WHERE id = 1
  AND stock > 0;

COMMIT;
```

The `FOR UPDATE` lock remains until the transaction commits or rolls back.

### What Does FOR UPDATE Do?

It locks selected rows against conflicting modifications or locking reads by other transactions.

It does not mean that all ordinary `SELECT` queries are blocked.

PostgreSQL's MVCC means ordinary readers can generally still read the last committed version.

---

## 7.6.6 Optimistic vs Pessimistic Locking

| Optimistic | Pessimistic |
|---|---|
| Assumes conflicts are uncommon | Assumes conflicts may happen |
| Doesn't lock when initially reading | Locks rows during the transaction |
| Checks version during update | Waits for lock availability |
| Conflict detected at update | Conflict handled through blocking/lock behavior |
| Better for low-conflict workloads | Better for contention-sensitive operations |

### Optimistic Memory Trick

```text
Read → Work → Check Version → Update
```

### Pessimistic Memory Trick

```text
Lock → Read → Work → Update → Unlock
```

### When Should You Use Pessimistic Locking?

Examples:

- Booking the last seat
- Inventory allocation
- Wallet debit
- Critical resource reservation
- Preventing simultaneous modifications to the same resource

But be careful: transactions should remain short. Long-held locks reduce throughput and increase the chance of deadlocks.

---

## 7.6.7 Database Locks

Database locks help control concurrent access to data.

### Common Lock Concepts

| Lock | Purpose |
|---|---|
| Shared lock | Allows compatible reads |
| Exclusive lock | Restricts conflicting access |
| Row lock | Locks individual rows |
| Table lock | Locks a table-level resource |
| Advisory lock | Application-defined coordination lock |

PostgreSQL has several lock modes. The simplified shared/exclusive model is useful for understanding the concept, but actual PostgreSQL locking is more nuanced.

### Row-Level Locking

```sql
SELECT *
FROM products
WHERE id = 1
FOR UPDATE;
```

Lock a specific row.

### Table-Level Locking

```sql
LOCK TABLE products
IN ACCESS EXCLUSIVE MODE;
```

Table locks can significantly impact concurrency, so use them carefully.

### Advisory Locks

PostgreSQL supports advisory locks, which are application-defined locks. They can be useful when you need to coordinate work that isn't naturally tied to locking a specific row.

For example, coordinating a job that should run once for a particular business key.

---

## 7.6.8 Deadlock

<badge>Interview favorite</badge>

A deadlock occurs when two or more transactions are waiting for locks held by each other, creating a cycle that prevents progress.

### Example

We have two records:

```text
Account A
Account B
```

**Transaction T1**

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 1;
```

T1 locks Account A.

Then T1 attempts:

```sql
UPDATE accounts
SET balance = balance + 100
WHERE id = 2;
```

**Transaction T2**

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 100
WHERE id = 2;
```

T2 locks Account B.

Then T2 attempts:

```sql
UPDATE accounts
SET balance = balance + 100
WHERE id = 1;
```

Now:

### Deadlock

```text
T1

Holds A

Waiting for B


T2

Holds B

Waiting for A
```

T1 waits for T2 → T2 waits for T1.

```text
T1 ── holds lock A ──►
                       |
                       ▼
                    needs B
                       ▲
                       |
T2 ── holds lock B ──►
                       |
                       ▼
                    needs A
```

PostgreSQL detects the deadlock and aborts one transaction.

Example error:

```text
ERROR: deadlock detected
SQLSTATE: 40P01
```

This is necessary because otherwise the transactions could wait indefinitely.

### How to Prevent Deadlocks?

#### 1. Lock Records in a Consistent Order

Bad:

```text
T1: Lock A → Lock B
T2: Lock B → Lock A
```

Better:

```text
T1: Lock A → Lock B
T2: Lock A → Lock B
```

Both transactions acquire locks in the same order.

#### 2. Keep Transactions Short

Avoid:

```text
BEGIN

Database query

Call external API

Wait 10 seconds

Update database

COMMIT
```

Instead, avoid holding database locks while waiting for external services.

#### 3. Avoid Unnecessary Locking

Don't use `FOR UPDATE` for every `SELECT`.

#### 4. Handle Deadlock Exceptions

A deadlock victim should generally retry the entire transaction if the operation is safe to retry.

---

## 7.6.9 Retry Strategy

Some database errors are temporary concurrency failures.

Examples:

- Deadlock
- Serialization failure
- Concurrent modification conflict

Your backend may retry the transaction.

### Retry Workflow

```text
Execute Transaction
       |
       ▼
  Successful?
   /     \
 YES      NO
  |        |
Return   Retry?
           |
           ▼
      Execute Again
```

### Example Java Retry Logic

```java
public void executeWithRetry(
        Runnable operation,
        int maxAttempts
) {

    int attempts = 0;

    while (attempts < maxAttempts) {

        try {
            operation.run();
            return;

        } catch (RuntimeException e) {

            attempts++;

            if (attempts >= maxAttempts) {
                throw e;
            }
        }
    }
}
```

This is a simplified retry example only.

In a real Spring application, transaction retry must be designed correctly. The retry should execute the operation in a new transaction, rather than retrying inside a transaction that has already been marked rollback-only.

### PostgreSQL SQLSTATE Codes to Remember

| SQLSTATE | Meaning |
|---|---|
| 40001 | Serialization failure |
| 40P01 | Deadlock detected |
| 23505 | Unique constraint violation |
| 23503 | Foreign key violation |

### Important

Not every database exception should be retried.

For example:

- Invalid SQL → retrying won't fix it.
- Invalid data → retrying won't fix it.
- Unique constraint violation → generally not a retryable concurrency failure.
- Deadlock → often retryable.
- Serialization failure → often retryable.

Also, retrying operations that call external APIs, charge payments, or send messages can cause duplicate side effects unless idempotency is designed correctly.

---

## 7.6.10 Implementing Optimistic Locking in Spring Boot

We'll use JDBC and `JdbcTemplate` first, because it helps you understand exactly what SQL the database executes.

### Table

```sql
CREATE TABLE inventory (
    id BIGINT PRIMARY KEY,
    product_name VARCHAR(100),
    stock INT NOT NULL,
    version BIGINT NOT NULL DEFAULT 0
);
```

### Repository

```java
@Repository
public class InventoryRepository {

    private final JdbcTemplate jdbcTemplate;

    public InventoryRepository(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public int decreaseStock(
            Long id,
            int quantity,
            Long currentVersion
    ) {

        String sql = """
            UPDATE inventory
            SET stock = stock - ?,
                version = version + 1
            WHERE id = ?
              AND version = ?
              AND stock >= ?
            """;

        return jdbcTemplate.update(
            sql,
            quantity,
            id,
            currentVersion,
            quantity
        );
    }
}
```

### Service

```java
@Service
public class InventoryService {

    private final InventoryRepository inventoryRepository;

    public InventoryService(
            InventoryRepository inventoryRepository
    ) {
        this.inventoryRepository = inventoryRepository;
    }

    public void purchase(
            Long productId,
            int quantity,
            Long version
    ) {

        int rowsUpdated =
            inventoryRepository.decreaseStock(
                productId,
                quantity,
                version
            );

        if (rowsUpdated == 0) {
            throw new IllegalStateException(
                "Stock insufficient or inventory was concurrently modified"
            );
        }
    }
}
```

### What Did We Protect?

Our SQL contains two conditions:

```sql
AND version = ?
AND stock >= ?
```

This protects against:

- Concurrent stale updates
- Overselling inventory

But remember, version-based optimistic locking is not always required for atomic counters. An atomic decrement with a stock condition may be sufficient.

Versioning is useful when you need to detect that a record changed since you read it.

---

## 7.6.11 Implementing Pessimistic Locking in Spring Boot

Using JDBC, you can explicitly lock a row.

### Repository

```java
@Repository
public class InventoryRepository {

    private final JdbcTemplate jdbcTemplate;

    public InventoryRepository(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public Integer findStockForUpdate(Long productId) {

        String sql = """
            SELECT stock
            FROM inventory
            WHERE id = ?
            FOR UPDATE
            """;

        return jdbcTemplate.queryForObject(
            sql,
            Integer.class,
            productId
        );
    }

    public int updateStock(
            Long productId,
            int quantity
    ) {

        String sql = """
            UPDATE inventory
            SET stock = stock - ?
            WHERE id = ?
            """;

        return jdbcTemplate.update(
            sql,
            quantity,
            productId
        );
    }
}
```

### Service

```java
@Service
public class InventoryService {

    private final InventoryRepository repository;

    public InventoryService(InventoryRepository repository) {
        this.repository = repository;
    }

    @Transactional
    public void purchase(
            Long productId,
            int quantity
    ) {

        Integer stock =
            repository.findStockForUpdate(productId);

        if (stock == null || stock < quantity) {
            throw new IllegalStateException(
                "Insufficient stock"
            );
        }

        repository.updateStock(
            productId,
            quantity
        );
    }
}
```

### Why Is @Transactional Important Here?

The row lock created by:

```sql
SELECT ... FOR UPDATE
```

is held until the transaction ends.

Without a proper transaction boundary, the lock may be released before the remaining operations are completed, making your logic incorrect.

### Proper Locking Workflow

```text
@Transactional
       |
       ▼
SELECT ... FOR UPDATE
       |
       ▼
Check stock
       |
       ▼
UPDATE stock
       |
       ▼
COMMIT
       |
       ▼
Lock released
```

This is the proper locking workflow.

---

## 7.6.12 Optimistic Locking with JPA — @Version

If you're using Spring Data JPA, optimistic locking is even easier.

### Entity

```java
@Entity
@Table(name = "products")
public class Product {

    @Id
    private Long id;

    private String name;

    private Integer stock;

    @Version
    private Long version;
}
```

`@Version` tells JPA to use the field for optimistic concurrency control.

When updating the entity, Hibernate generates SQL conceptually similar to:

```sql
UPDATE products
SET stock = ?,
    version = version + 1
WHERE id = ?
  AND version = ?;
```

If another transaction already modified the row, the update affects zero rows and JPA throws an optimistic locking exception, commonly:

```text
OptimisticLockException
```

Or through Spring's exception translation:

```text
ObjectOptimisticLockingFailureException
```

### Remember

```text
JDBC
    → Implement version check manually

JPA
    → @Version
```

---

# 7.6 — Quick Revision Sheet

## Concurrency Cheat Sheet

| Concept | Meaning |
|---|---|
| Lost Update | One concurrent update overwrites another |
| Race Condition | Outcome depends on concurrent execution timing |
| Optimistic Locking | Detect conflict using a version or condition |
| Pessimistic Locking | Lock resource first; other conflicting transactions wait |
| FOR UPDATE | Lock selected database rows for update |
| Deadlock | Transactions wait for locks held by each other |
| Atomic SQL | Perform calculations directly in a single database operation |
| Retry | Retry transient transaction failures in a new transaction |
| JPA @Version | Automatic optimistic locking support |

## Interview Comparison

| Scenario | Recommended Technique |
|---|---|
| Increment/decrement counter | Atomic SQL |
| Inventory decrement | Atomic SQL with condition |
| User profile edit | Optimistic locking |
| Concurrent editing of same configuration | Optimistic locking |
| Booking scarce resources | Pessimistic lock or atomic reservation |
| Money transfer | Transaction + appropriate locking |
| Deadlock | Consistent lock ordering + retry |
| Serialization failure | Retry entire transaction |

---

# Test Yourself

### 1. Two users read stock 10 and both write stock 9. Final stock becomes 9 instead of 8. What is this?

- [ ] Dirty read
- [ ] Lost update
- [ ] Phantom read
- [ ] Deadlock

### 2. Which locking strategy detects conflicts using a version column?

- [ ] Pessimistic locking
- [ ] Table locking
- [ ] Optimistic locking
- [ ] Shared locking

### 3. What does SELECT ... FOR UPDATE do?

- [ ] Deletes the selected rows
- [ ] Locks selected rows for conflicting operations
- [ ] Creates an index
- [ ] Commits the transaction

### 4. T1 holds Lock A and waits for Lock B. T2 holds Lock B and waits for Lock A. What happened?

- [ ] Dirty read
- [ ] Lost update
- [ ] Deadlock
- [ ] Phantom read

### 5. For a simple stock decrement, which approach is usually best?

- [ ] Read stock into Java and update later
- [ ] Atomic UPDATE stock = stock - 1 with a stock condition
- [ ] Use Thread.sleep()
- [ ] Disable transactions

---

# Next Topic in Your Backend Revision

## 7.7 — JDBC Batch Processing & Performance

Topics:

- Batch insert
- `addBatch()`
- `executeBatch()`
- Batch updates
- Batch size
- JDBC performance
- N+1 problem introduction
- Large dataset processing
- `fetchSize`
- Database round trips
- When to use batch processing

This is a natural continuation after JDBC, transactions, connection pooling, and concurrency.