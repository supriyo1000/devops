# Phase 7.7 — JDBC Batch Processing & Performance
Java + PostgreSQL + Employee Project

Today, we'll learn how to efficiently insert, update, and process large amounts of employee data using pure Java JDBC, without Spring Boot.

This topic is important because inserting 10,000 employees one by one is very different from inserting 10,000 employees using JDBC batching.

## 1. What problem does batch processing solve?
Suppose your Employee project needs to insert 10,000 employees into PostgreSQL.

### Approach 1 — Normal JDBC insert
You execute one SQL statement for each employee.

```
for (Employee employee : employees) {

    PreparedStatement statement =
        connection.prepareStatement(
            "INSERT INTO employees (name, email, salary) VALUES (?, ?, ?)"
        );

    statement.setString(1, employee.getName());
    statement.setString(2, employee.getEmail());
    statement.setBigDecimal(3, employee.getSalary());

    statement.executeUpdate();
}
```

What happens?
10,000 employees = 10,000 database executions

```
Java
 │
 ├── INSERT Employee 1 ──→ PostgreSQL
 ├── INSERT Employee 2 ──→ PostgreSQL
 ├── INSERT Employee 3 ──→ PostgreSQL
 ├── INSERT Employee 4 ──→ PostgreSQL
 ├── INSERT Employee 5 ──→ PostgreSQL
 │
 └── ...
      10,000 database executions
```


This can be slow because of the overhead associated with repeatedly executing statements, network communication, protocol processing, and database execution.

The problem is not necessarily that the SQL query itself is slow.

The problem is the number of individual executions.

## 2. What is JDBC Batch Processing?
Batch processing allows you to prepare multiple operations and submit them together through JDBC.

Instead of executing:

```
INSERT Employee 1
INSERT Employee 2
INSERT Employee 3
INSERT Employee 4
INSERT Employee 5
```

You prepare a batch:

```
Prepare Employee 1
Prepare Employee 2
Prepare Employee 3
Prepare Employee 4
Prepare Employee 5
         │
         ▼
    executeBatch()
         │
         ▼
   PostgreSQL processes
      the batch
```

The primary JDBC methods are:

```
Method	          Purpose

addBatch()	      Add a prepared operation to a batch
executeBatch()	  Execute the accumulated batch
clearBatch()	  Clear the batch
executeUpdate()	  Execute one update operation
```

## 3. The Employee project we will use
Our table:

```
CREATE TABLE employees (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL,
    salary NUMERIC(10, 2) NOT NULL,
    department VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

Our Java model:

```
package com.example.jdbc.model;

import java.math.BigDecimal;

public class Employee {

    private String name;
    private String email;
    private BigDecimal salary;
    private String department;

    public Employee(
            String name,
            String email,
            BigDecimal salary,
            String department
    ) {
        this.name = name;
        this.email = email;
        this.salary = salary;
        this.department = department;
    }

    public String getName() {
        return name;
    }

    public String getEmail() {
        return email;
    }

    public BigDecimal getSalary() {
        return salary;
    }

    public String getDepartment() {
        return department;
    }
}
```

Notice that we use BigDecimal for salary rather than double.

Why?

Floating-point types such as double can introduce precision problems when representing decimal monetary values.

## 4. Understanding addBatch() and executeBatch()
Before writing the complete implementation, let's understand the mechanism.
```
PreparedStatement statement = connection.prepareStatement(
    "INSERT INTO employees (name, email, salary, department) VALUES (?, ?, ?, ?)"
);
```

This creates a reusable SQL statement.

We then set parameters for each employee.

#### Employee 1
```
statement.setString(1, "Rahul");
statement.setString(2, "rahul@example.com");
statement.setBigDecimal(3, new BigDecimal("50000"));
statement.setString(4, "IT");

statement.addBatch();
```

#### Employee 2
```
statement.setString(1, "Priya");
statement.setString(2, "priya@example.com");
statement.setBigDecimal(3, new BigDecimal("60000"));
statement.setString(4, "HR");

statement.addBatch();
```

#### Employee 3
```
statement.setString(1, "Amit");
statement.setString(2, "amit@example.com");
statement.setBigDecimal(3, new BigDecimal("45000"));
statement.setString(4, "Finance");

statement.addBatch();
```

At this point, we have not executed the individual operations using executeUpdate().

We've accumulated the batch.
Batch contents
```
Batch #1	Rahul — 50000
Batch #2	Priya — 60000
Batch #3	Amit — 45000
```

Then:
```
int[] results = statement.executeBatch();
```

This executes the queued batch operations.

## 5. Implement batch insert in EmployeeDAO
Let's update our existing DAO architecture.

```
jdbc-project/
│
├── config/
│   └── DatabaseConfig.java
│
├── model/
│   └── Employee.java
│
├── dao/
│   └── EmployeeDAO.java
│
├── service/
│   └── EmployeeService.java
│
└── Main.java
```


### EmployeeDAO — batch insert

```
package com.example.jdbc.dao;

import com.example.jdbc.config.DatabaseConfig;
import com.example.jdbc.model.Employee;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.util.List;

public class EmployeeDAO {

    public int[] batchInsert(List<Employee> employees)
            throws SQLException {

        String sql = """
            INSERT INTO employees
                (name, email, salary, department)
            VALUES (?, ?, ?, ?)
            """;

        try (
            Connection connection = DatabaseConfig.getConnection();
            PreparedStatement statement =
                connection.prepareStatement(sql)
        ) {

            for (Employee employee : employees) {

                statement.setString(1, employee.getName());
                statement.setString(2, employee.getEmail());
                statement.setBigDecimal(3, employee.getSalary());
                statement.setString(4, employee.getDepartment());

                statement.addBatch();
            }

            return statement.executeBatch();
        }
    }
}
```

What is happening?

#### Execution flow

1. Create SQL PreparedStatement

2. Loop through employee list

3. Set parameters

4. Call addBatch()

5. Repeat for every employee

6. executeBatch()

7. Return execution results


Why reuse the same PreparedStatement?
Instead of creating a new PreparedStatement for every employee:
```
for (Employee employee : employees) {

    PreparedStatement statement =
        connection.prepareStatement(sql);

    // Set values
    statement.executeUpdate();
}
```

We create it once:
```
PreparedStatement statement =
    connection.prepareStatement(sql);
```

Then reuse it throughout the loop.

This reduces statement creation overhead and enables batch processing.

## 6. What does executeBatch() return?
```
int[] results = statement.executeBatch();
```

It returns an array containing execution results for the batch commands.

#### For example:
[1, 1, 1, 1, 1]


This often means five operations each affected one row.

For JDBC batch execution, results can also include special constants:

```
Result	Meaning
1	One row affected
0	No rows affected

Statement.SUCCESS_NO_INFO	Successful operation, affected row count unknown
Statement.EXECUTE_FAILED	Operation failed
```

Example:

```
for (int result : results) {

    if (result == Statement.SUCCESS_NO_INFO) {
        System.out.println("Success, count unknown");
    } else if (result == Statement.EXECUTE_FAILED) {
        System.out.println("Failed");
    } else {
        System.out.println("Rows affected: " + result);
    }
}
```


## 7. Batch size — a very important performance concept
Suppose you have 100,000 employees.

Should you add all 100,000 records to one batch?

Usually, you should consider breaking them into smaller batches.

For example:
```
100,000 Employees

Batch 1 → 1,000
Batch 2 → 1,000
Batch 3 → 1,000
...
Batch 100 → 1,000
```

Why?

A very large batch can consume:
- Java memory
- JDBC driver memory
- Database resources
- Network buffers
- Transaction resources


The ideal batch size depends on:
- Record size
- Database
- Network latency
- JDBC driver
- Available memory
- Indexes
- Constraints
- Transaction duration
There is no universally correct batch size.

For practice, we will start with 1,000 records per batch and benchmark it.

## 8. Production-style batch insert with batch size
Now let's improve our DAO.

We will:
1. Accept a list of employees.
2. Create one PreparedStatement.
3. Accumulate employees in a batch.
4. Execute when the batch reaches the configured size.
5. Execute the remaining records at the end.
6. Return the number of operations attempted through batch result counts.

```
package com.example.jdbc.dao;

import com.example.jdbc.config.DatabaseConfig;
import com.example.jdbc.model.Employee;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.List;

public class EmployeeDAO {

    private static final int BATCH_SIZE = 1000;

    public long batchInsert(
            List<Employee> employees
    ) throws SQLException {

        String sql = """
            INSERT INTO employees
                (name, email, salary, department)
            VALUES (?, ?, ?, ?)
            """;

        long totalInserted = 0;

        try (
            Connection connection = DatabaseConfig.getConnection();
            PreparedStatement statement =
                connection.prepareStatement(sql)
        ) {

            int count = 0;

            for (Employee employee : employees) {

                statement.setString(1, employee.getName());
                statement.setString(2, employee.getEmail());
                statement.setBigDecimal(3, employee.getSalary());
                statement.setString(4, employee.getDepartment());

                statement.addBatch();

                count++;

                if (count % BATCH_SIZE == 0) {

                    int[] results = statement.executeBatch();

                    totalInserted += countSuccessfulResults(results);

                    statement.clearBatch();
                }
            }

            // Execute remaining records
            if (count % BATCH_SIZE != 0) {

                int[] results = statement.executeBatch();

                totalInserted += countSuccessfulResults(results);

                statement.clearBatch();
            }
        }

        return totalInserted;
    }

    private long countSuccessfulResults(int[] results) {

        long count = 0;

        for (int result : results) {

            if (result >= 0 ||
                result == Statement.SUCCESS_NO_INFO) {

                count++;
            }
        }

        return count;
    }
}
```

Example
Suppose you insert 2,500 employees.

```
Batch 1
Executed

1,000 employees

Batch 2
Executed

1,000 employees

Batch 3
Executed

500 employees
```
This is the basic pattern used when processing larger datasets.

Note: The example above executes batches but does not explicitly start a transaction covering all batches. Depending on the connection's auto-commit behavior and driver configuration, batches may not be atomic as a whole. We will handle that correctly next.

## 9. Batch processing and transactions
This is where the topics you learned earlier come together.

Suppose you are inserting 10,000 employees.

```
You execute:
statement.executeBatch();

Then:
statement.executeBatch();

Then:
statement.executeBatch();
```

### What happens if batch 3 fails?

Without an explicit transaction covering the work, the earlier successful batches may already be committed.

You may end up with:
```
Batch 1 → SUCCESS → COMMITTED
Batch 2 → SUCCESS → COMMITTED
Batch 3 → FAILED
Batch 4 → NOT EXECUTED

Database:
2,000 employees inserted
8,000 employees missing
```

This might be acceptable for some bulk-import jobs—but not all.


#### Option A: One transaction for the entire import

```
connection.setAutoCommit(false);

try {

    // Batch 1
    statement.executeBatch();

    // Batch 2
    statement.executeBatch();

    // Batch 3
    statement.executeBatch();

    connection.commit();

} catch (SQLException ex) {

    connection.rollback();
    throw ex;

} finally {

    connection.setAutoCommit(true);
}
```

Atomic import
```
BEGIN
   Batch 1
   Batch 2
   Batch 3
   Batch 4
COMMIT
```

If an error occurs:
```
BEGIN
   Batch 1
   Batch 2
   Batch 3 → ERROR
ROLLBACK
```

All changes made within the transaction are rolled back.

Advantages
- All employees inserted or none.
- Strong consistency.
- Easy to reason about.
Disadvantages
- Long transaction.
- Locks/resources may be held longer.
- Large transaction can create substantial database overhead.
- Rollback can be expensive.

#### Option B: Commit each batch

Batch 1 → BEGIN → INSERT 1000 → COMMIT

Batch 2 → BEGIN → INSERT 1000 → COMMIT

Batch 3 → BEGIN → INSERT 1000 → COMMIT


Advantages
- Smaller transactions.
- Less transaction resource usage.
- Failure doesn't require rolling back the entire import.


Disadvantages
- Import may be partially completed.
- You need a restart/resume strategy.
- You need to track which records were processed.


#### When should you choose?


Requirement	Strategy
All-or-nothing import	Single transaction
Huge import	Commit per batch may be preferable
Resumable import	Commit batches + checkpoint
Financial consistency	Carefully designed atomic transactions
Data migration	Often chunked batches with tracking
Millions of records	Chunking and controlled transactions


## 10. Complete transactional batch insert
This version demonstrates one transaction for the complete operation.

```
public long batchInsertTransactional(
        List<Employee> employees
) throws SQLException {

    String sql = """
        INSERT INTO employees
            (name, email, salary, department)
        VALUES (?, ?, ?, ?)
        """;

    long inserted = 0;

    try (
        Connection connection = DatabaseConfig.getConnection();
        PreparedStatement statement =
            connection.prepareStatement(sql)
    ) {

        connection.setAutoCommit(false);

        try {

            int count = 0;

            for (Employee employee : employees) {

                statement.setString(1, employee.getName());
                statement.setString(2, employee.getEmail());
                statement.setBigDecimal(3, employee.getSalary());
                statement.setString(4, employee.getDepartment());

                statement.addBatch();

                count++;

                if (count % BATCH_SIZE == 0) {

                    int[] results = statement.executeBatch();

                    inserted += countSuccessfulResults(results);

                    statement.clearBatch();
                }
            }

            if (count % BATCH_SIZE != 0) {

                int[] results = statement.executeBatch();

                inserted += countSuccessfulResults(results);

                statement.clearBatch();
            }

            connection.commit();

            return inserted;

        } catch (SQLException ex) {

            connection.rollback();

            throw ex;

        } finally {

            connection.setAutoCommit(true);
        }
    }
}
```

A production implementation should also consider failures while restoring auto-commit, connection cleanup, rollback failures, and batch exceptions. This example is for learning transaction structure.

## 11. What is the difference between batch size and transaction size?
Interviewers sometimes ask this.

These are two separate concepts.

#### Batch size
How many statements are grouped for one batch execution?

Example:
executeBatch() every 1,000 statements

#### Transaction size
How many database operations are included in one transaction before commit?

Example:
Commit after every 5 batches

1,000 × 5 = 5,000 records per transaction



You can have:
```
Batch size = 1,000
Transaction size = 10,000

10 batches
        ↓
1 COMMIT

Or:

Batch size = 1,000
Transaction size = 1,000

1 batch
        ↓
1 COMMIT
```

Batch size controls execution grouping.

Transaction size controls atomicity and commit behavior.

## 12. Why JDBC batching improves performance
Let's compare conceptual database calls.

Normal inserts

```
Java
 ├─ INSERT 1 → DB
 ├─ INSERT 2 → DB
 ├─ INSERT 3 → DB
 ├─ INSERT 4 → DB
 ├─ INSERT 5 → DB
 └─ ...
```

Batched inserts
```
Java
 ├─ Prepare INSERT
 ├─ Add 1
 ├─ Add 2
 ├─ Add 3
 ├─ Add 4
 ├─ Add 5
 │
 └─ executeBatch() → DB
```

Batching can reduce overhead by reducing the cost associated with repeated statement execution and network communication.

But remember:
Batch processing doesn't guarantee that the database will execute every SQL statement as one single SQL command.

The JDBC driver and database determine how the batch is sent and processed. PostgreSQL driver configuration can affect batch performance as well.

## 13. JDBC performance — the important principles

### Principle 1 — Reuse PreparedStatement
Bad:
```
for (Employee employee : employees) {

    PreparedStatement statement =
        connection.prepareStatement(sql);

    // Execute
}
```

Better:

```
PreparedStatement statement =
    connection.prepareStatement(sql);

for (Employee employee : employees) {

    // Set parameters

    statement.addBatch();
}
```


### Principle 2 — Avoid unnecessary database round trips
Bad approach:

```
for (Long employeeId : employeeIds) {

    Employee employee = findById(employeeId);

    updateEmployee(employee);
}
```

If there are 10,000 employees:
- 10,000 SELECT operations
- 10,000 UPDATE operations


That's potentially 20,000 database operations.

Think about:
- Batch queries
- Bulk updates
- Joins
- Fetching data in larger groups
- Database-side operations

### Principle 3 — Do not fetch unnecessary columns
Bad:

```
SELECT *
FROM employees;
```

If you only need names:

Better:
```
SELECT name
FROM employees;
```

This reduces:
- Data transfer
- Memory usage
- Processing overhead

### Principle 4 — Avoid unnecessary Java processing
Suppose you need to increase everyone's salary by 5%.

You could:
1. Read all employees.
2. Loop in Java.
3. Calculate salary.
4. Update every employee individually.

Or:

UPDATE employees
SET salary = salary * 1.05;


The database is designed to perform data operations efficiently.

For data manipulation, evaluate whether SQL can perform the operation directly.

### Principle 5 — Use indexes appropriately
Consider:

```
SELECT *
FROM employees
WHERE email = 'rahul@example.com';
```

If email is indexed, the database may be able to find the employee efficiently without scanning the entire table.

But indexes also cost resources.

Indexes can:
- Improve reads.
- Slow inserts.
- Slow updates to indexed columns.
- Consume disk space.
Don't create an index on every column.

### Principle 6 — Manage connections correctly
Never do this for every operation unnecessarily:
DriverManager.getConnection(...);


A connection involves database and network resources.

Use:
```
DataSource
    ↓
Connection Pool
    ↓
Reusable Connections
```

In our earlier lessons, we covered HikariCP.

## 14. Large datasets — what happens if you load everything?
Imagine the database contains 1 million employees.

You run:
```
List<Employee> employees = findAll();
```

The implementation loads everything into Java memory.
Potential memory problem
PostgreSQL

1,000,000 records


JDBC ResultSet

Java List
Potential high heap usage / OutOfMemoryError

For large data, don't automatically use:
List<Employee> employees = findAll();


Instead, consider:
- Pagination
- Streaming
- Cursor-based processing
- JDBC fetch size
- Chunk processing

## 15. Pagination — process employees in chunks
Instead of fetching one million employees:

```
SELECT *
FROM employees;
```

Fetch 1,000 at a time.
```
SELECT *
FROM employees
ORDER BY id
LIMIT 1000 OFFSET 0;
```

Then:
```
SELECT *
FROM employees
ORDER BY id
LIMIT 1000 OFFSET 1000;
```

Then:
```
SELECT *
FROM employees
ORDER BY id
LIMIT 1000 OFFSET 2000;
```

#### Java:
```
int pageSize = 1000;
int offset = 0;

while (true) {

    List<Employee> employees =
        employeeDAO.findEmployees(pageSize, offset);

    if (employees.isEmpty()) {
        break;
    }

    processEmployees(employees);

    offset += pageSize;
}
```

OFFSET pagination becomes increasingly expensive for large offsets. Later, we'll learn keyset pagination, which is often more efficient for processing large tables.

## 16. JDBC fetchSize
This is a particularly important topic for large result sets.

Consider:
```
PreparedStatement statement =
    connection.prepareStatement(
        "SELECT * FROM employees"
    );

ResultSet rs = statement.executeQuery();
```

Depending on driver configuration and behavior, you might receive or buffer a large result set in memory.

fetchSize provides a hint to the JDBC driver about how many rows should be fetched from the database at a time.
#### statement.setFetchSize(500);


Conceptually:
```
Database
    │
    │ 500 rows
    ▼
Java
    │
    │ Process rows
    ▼
Java
    │
    │ Request next 500
    ▼
Database
```

Instead of trying to materialize the entire result set at once, the driver can retrieve data in chunks.


Example

```
public void processAllEmployees() throws SQLException {

    String sql = "SELECT id, name, email, salary FROM employees";

    try (
        Connection connection = DatabaseConfig.getConnection();
        PreparedStatement statement =
            connection.prepareStatement(sql)
    ) {

        statement.setFetchSize(500);

        try (ResultSet rs = statement.executeQuery()) {

            while (rs.next()) {

                long id = rs.getLong("id");

                String name = rs.getString("name");

                String email = rs.getString("email");

                // Process record
                System.out.println(id + " " + name + " " + email);
            }
        }
    }
}
```

#### fetchSize vs batch size

Batch size = how many write operations you group together.

Fetch size = how many read rows you ask the driver to retrieve at a time.

For PostgreSQL, cursor-based fetching behavior has specific requirements—commonly, a non-auto-commit connection and a positive fetch size are used to enable cursor-backed fetching with the PostgreSQL JDBC driver. We will cover this in detail during the streaming lesson.

## 17. EmployeeService — generate and insert employees
Let's make the example practical. We will generate employee records and insert them in batches.

```
package com.example.jdbc.service;

import com.example.jdbc.dao.EmployeeDAO;
import com.example.jdbc.model.Employee;

import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.List;

public class EmployeeService {

    private final EmployeeDAO employeeDAO = new EmployeeDAO();

    public long generateEmployees(int total) {

        List<Employee> employees = new ArrayList<>(total);

        for (int i = 1; i <= total; i++) {

            Employee employee = new Employee(
                "Employee " + i,
                "employee" + i + "@example.com",
                new BigDecimal("50000.00"),
                "IT"
            );

            employees.add(employee);
        }

        try {
            return employeeDAO.batchInsertTransactional(employees);
        } catch (Exception ex) {
            throw new RuntimeException(
                "Failed to insert employees",
                ex
            );
        }
    }
}
```

#### Main.java

```
package com.example.jdbc;

import com.example.jdbc.service.EmployeeService;

public class Main {

    public static void main(String[] args) {

        EmployeeService employeeService = new EmployeeService();

        long inserted =
            employeeService.generateEmployees(10000);

        System.out.println(
            "Employees inserted: " + inserted
        );
    }
}
```

Run it:
Employees inserted: 10000


Important: Run the test against a table that doesn't already contain those emails, because email is unique.

## 18. Benchmark — normal insert vs batch insert
Now the most useful practical exercise.

We will compare three approaches.
Performance experiment
Approach A
Normal JDBC

Insert one record at a time using executeUpdate().

Approach B
JDBC Batch

Accumulate records using addBatch() and execute using executeBatch().

Approach C
JDBC Batch + Transactions

Batch inserts inside a controlled transaction.

Benchmark methodology
Test with:
- 1,000 employees
- 10,000 employees
- 50,000 employees


Keep the following identical for each test:
- Same PostgreSQL instance
- Same table structure
- Same indexes
- Same dataset
- Same connection pool configuration
- Same transaction policy, when comparing batch execution alone


#### Use System.nanoTime() for measuring duration.

```
long start = System.nanoTime();

// Execute database operation

long end = System.nanoTime();

long durationMs = (end - start) / 1_000_000;

System.out.println("Execution time: " + durationMs + " ms");
```

Important benchmarking rule
Do not compare:

- Normal inserts with autocommit enabled

against:

- Batch inserts with one transaction

and conclude that batching alone caused the difference.

You have changed two variables:
1. Batch execution
2. Transaction behavior
For a fair benchmark, first compare both methods under the same transaction strategy. Then separately test the impact of transaction boundaries.

## 19. PostgreSQL-specific batch optimization
The PostgreSQL JDBC driver supports a configuration option that can improve performance for certain batch INSERT workloads.
```
reWriteBatchedInserts=true
```

Example JDBC URL:
jdbc:postgresql://localhost:5432/jdbc_demo?reWriteBatchedInserts=true


With this enabled, the driver may rewrite compatible batched inserts into a multi-row INSERT statement.

Instead of sending the conceptual operations:
INSERT INTO employees (...) VALUES (...);
INSERT INTO employees (...) VALUES (...);
INSERT INTO employees (...) VALUES (...);


The driver may send a rewritten statement like:

```
INSERT INTO employees
    (name, email, salary, department)
VALUES
    ('Employee 1', 'employee1@example.com', 50000, 'IT'),
    ('Employee 2', 'employee2@example.com', 50000, 'IT'),
    ('Employee 3', 'employee3@example.com', 50000, 'IT');
```

This is a driver optimization—not a JDBC guarantee. Test it with your workload and driver version.

## 20. What is the N+1 problem? (Introduction)
This is a problem you will later encounter in JDBC, Hibernate, and JPA.

Suppose you have:

```
Departments
    │
    ├── IT
    ├── HR
    └── Finance

Employees
    │
    ├── Employee 1 → IT
    ├── Employee 2 → IT
    ├── Employee 3 → HR
    ├── Employee 4 → Finance
    └── ...
```

You execute:

Query 1

```
SELECT id, name
FROM departments;
```

It returns 10 departments.

Then, for every department:

```
SELECT *
FROM employees
WHERE department_id = ?;
```

If there are 10 departments:
1 query → departments

10 queries → employees of each department

Total = 11 queries


This is the N+1 query problem.
#### Why this matters
1 + N database queries
```
Query 1
    ↓
Get 10 departments
    ↓
Query 2 → IT employees
Query 3 → HR employees
Query 4 → Finance employees
...
Query 11
```

This might be fine with 10 departments but becomes costly when there are hundreds or thousands.

A better SQL approach might be:

```
SELECT
    d.id AS department_id,
    d.name AS department_name,
    e.id AS employee_id,
    e.name AS employee_name
FROM departments d
LEFT JOIN employees e
    ON e.department_id = d.id;
```

This returns data using one SQL query.

We will study the N+1 problem in depth later when we reach JDBC optimization and Hibernate performance.

## 21. When should you use JDBC batch processing?
|Use case|	Batch processing|
|--------|------------------|
|Insert 1 employee|	Usually unnecessary|
|Insert 10 employees|	Optional|
|Insert 1,000 employees|	Usually useful|
|Import CSV with 100,000 records|	Strong candidate|
|ETL data pipeline|	Strong candidate|
|Database migration|	Strong candidate|
|Update many records|	Strong candidate|
|Insert logs one at a time|	Depends on throughput|
|Processing events|	Often useful|
|Bulk data synchronization|	Strong candidate|

Batching is not automatically better for every query.

For a single record, ordinary executeUpdate() is completely fine.

## 22. Java JDBC performance checklist
When you write database code, remember these rules.
Database performance checklist
Connection management
- [x] Reuse connections through DataSource/pooling
- [ ] Never leak connections
- [ ] Keep transactions reasonably short

SQL execution
- [ ] Use PreparedStatement
- [ ] Avoid unnecessary SQL executions
- [x] Use batching for bulk writes
- [ ] Prefer database-side operations when appropriate

Data retrieval
- [ ] Avoid SELECT *
- [ ] Avoid loading unnecessary records
- [x] Use pagination where appropriate
- [ ] Use fetchSize for large result sets where supported
- [ ] Avoid loading huge datasets into Java collections

Transactions
- [ ] Choose transaction boundaries carefully
- [x] Understand commit and rollback
- [x] Avoid excessively long transactions
- [ ] Understand the difference between batch size and transaction size

Performance
- [x] Measure execution time
- [ ] Inspect execution plans
- [x] Use appropriate indexes
- [ ] Benchmark before optimizing
- [ ] Monitor connection pools


## 23. Questions — Batch Processing
#### Q1. What is JDBC batch processing?
JDBC batch processing allows multiple SQL operations to be accumulated and executed together using addBatch() and executeBatch(), reducing repeated execution overhead.
#### Q2. What is the difference between executeUpdate() and executeBatch()?
executeUpdate() executes one SQL operation and returns the affected row count.

executeBatch() executes accumulated operations and returns an integer array of results.
#### Q3. What is the purpose of addBatch()?
It adds the current parameterized statement execution to the batch queue without executing it immediately.
#### Q4. Does JDBC batching guarantee one database round trip?
No. It reduces execution overhead, but the actual communication and execution behavior depend on the JDBC driver and database.
#### Q5. What is batch size?
The number of operations grouped into one batch execution.
#### Q6. What is the difference between batch size and transaction size?
- Batch size controls how many operations are submitted together.
- Transaction size controls how many operations are committed or rolled back together.
#### Q7. Why not insert 1 million rows in one batch?
A huge batch can consume memory and database resources, create long-running transactions, and increase failure recovery costs. Chunking is usually more manageable.
#### Q8. What is fetchSize?
A JDBC hint for how many rows the driver should fetch at a time when retrieving query results.
#### Q9. What is the difference between fetchSize and batchSize?
- fetchSize → retrieving data.
- batchSize → sending data operations.
#### Q10. What is the N+1 query problem?
Executing one query to retrieve parent records, then one additional query for each parent to retrieve related records, causing excessive database calls.