# Phase 7.8 — JDBC Exception Handling & SQL Error Management
Pure Java + JDBC + PostgreSQL | Employee Project

In the previous lessons, we learned how to:
- Connect Java to PostgreSQL.
- Execute SQL using PreparedStatement.
- Insert, update, delete, and retrieve employees.
- Use HikariCP.
- Manage transactions.
- Handle batches.
- Improve performance.
But in a real application, database operations don't always succeed.

For example:
- The employee email already exists.
- An employee ID doesn't exist.
- PostgreSQL is down.
- The connection times out.
- The SQL query contains an error.
- A foreign key constraint is violated.
- Two transactions deadlock.
- A transaction fails during batch processing.
Today, we'll learn how to identify these errors, understand them, and build a proper exception-handling structure in Java.

## 1. What happens when a JDBC operation fails?
Suppose you execute:

```
String sql = """
    INSERT INTO employees (name, email, salary, department)
    VALUES (?, ?, ?, ?)
    """;
```

You try inserting:
```
Name: Rahul
Email: rahul@example.com
Salary: 50000
Department: IT
```

But the email already exists in the database.

PostgreSQL rejects the operation.

The flow looks like this:

```
Java Application
EmployeeDAO executes INSERT

PostgreSQL
Detects duplicate email

SQL Error
Unique constraint violation

SQLException
JDBC exposes database failure to Java

Java Exception Handling
Catch, classify, handle, or propagate
```
The database is responsible for detecting the database error.

JDBC communicates that failure to your Java application through exceptions, typically subclasses or instances of SQLException.

## 2. What is SQLException?
SQLException is the primary checked exception used by JDBC for database-related failures.

```
import java.sql.SQLException;


Example:
try {

    Connection connection = dataSource.getConnection();

} catch (SQLException ex) {

    System.out.println("Database error");
}
```

SQLException is a checked exception.

That means Java generally requires you to either:
1. Catch it using try-catch, or
2. Declare it using throws.

#### Option 1 — Catch it

```
public void saveEmployee() {

    try {
        // JDBC operations

    } catch (SQLException ex) {

        System.out.println("Database operation failed");
    }
}
```

### Option 2 — Propagate it

```
public void saveEmployee() throws SQLException {

    // JDBC operations
}
```

The exception can travel through the call stack.

```
Main
  ↓
EmployeeService
  ↓
EmployeeDAO
  ↓
PostgreSQL
  │
  └── SQLException
       ↑
   EmployeeDAO
       ↑
 EmployeeService
       ↑
     Main
```

This is called exception propagation.

## 3. Understanding SQLException properly
SQLException provides more than just a message.

#### Important methods:

|Method|	Purpose|
|------|-----------|
|getMessage()	|Error message|
|getSQLState()	|Standardized error category/code|
|getErrorCode()|	Database-specific error code|
|getCause()|	Underlying cause|
|getNextException()|	Next chained SQL exception|

Example:
```
catch (SQLException ex) {

    System.out.println("Message: " + ex.getMessage());

    System.out.println("SQLState: " + ex.getSQLState());

    System.out.println("Error Code: " + ex.getErrorCode());
}
```

Example PostgreSQL duplicate email error might look conceptually like:
#### Message:
duplicate key value violates unique constraint "employees_email_key"

#### SQLState:
23505

#### Error Code:
0


PostgreSQL JDBC generally exposes PostgreSQL SQLState values and may expose a vendor error code that is not the same as SQLState.

Do not depend only on the human-readable error message. Messages can change, vary by language, or include implementation details.

## 4. SQLState — identify the category of database error
SQLState is a standardized code that helps identify the type of SQL/database error.

Instead of checking:
```
if (ex.getMessage().contains("duplicate")) {
    // ...
}


We can check:
if ("23505".equals(ex.getSQLState())) {
    // Unique constraint violation
}
```

This is more reliable.
### Important PostgreSQL SQLState codes

|SQLState|	Meaning|	Example|
|--------|---------|-----------|
|23505|	Unique violation|	Duplicate email|
|23503|	Foreign key violation|	Referenced employee/dept missing|
|23502|	Not-null violation|	Required value missing|
|23514|	Check constraint| violation	Salary check failed|
|22001|	String too long|	Name exceeds column length|
|22003|	Numeric value out of range|	Salary exceeds allowed range|
|40001|	Serialization failure|	Serializable transaction conflict|
|40P01|	Deadlock detected|	Transactions waiting on each other|
|08000|	Connection exception|	Connection-related failure|
|08003|	Connection does not exist|	Connection unavailable|
|08006|	Connection failure|	Connection lost|
|57014|	Query canceled|	Query timeout/cancellation|
|42601|	Syntax error|	Invalid SQL syntax|
|42P01|	Undefined table	|Table doesn't exist|
|42703|	Undefined column|	Column doesn't exist|

SQLState codes are much better than parsing exception messages, but always verify the code's meaning for your database and driver.

## 5. Example: duplicate employee email
Our database contains:

```
CREATE TABLE employees (
    id BIGSERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE NOT NULL,
    salary NUMERIC(10, 2) NOT NULL,
    department VARCHAR(100)
);
```

Suppose this employee already exists:
```
Rahul
rahul@example.com
50000
IT
```

Now we try to insert the same email again.
```
Employee employee = new Employee(
    "Rahul",
    "rahul@example.com",
    new BigDecimal("60000"),
    "HR"
);
```

The database rejects it because email has a unique constraint.
#### Simple exception handling

```
try {

    employeeDAO.save(employee);

} catch (SQLException ex) {

    if ("23505".equals(ex.getSQLState())) {

        System.out.println("Employee email already exists.");

    } else {

        System.out.println("Database operation failed.");
    }
}
```

This works, but there is a problem.

If every service and every controller checks SQLState individually, the application becomes difficult to maintain.

We need a better architecture.

## 6. Why we need custom exceptions
Imagine your application has:

- EmployeeDAO
- DepartmentDAO
- ProjectDAO
- CustomerDAO
- OrderDAO


Every DAO throws SQLException.

Your service code starts looking like this:

```
try {
    employeeDAO.save(employee);

} catch (SQLException ex) {

    if ("23505".equals(ex.getSQLState())) {
        // Duplicate
    }

    if ("23503".equals(ex.getSQLState())) {
        // Foreign key
    }

    if ("08006".equals(ex.getSQLState())) {
        // Connection failure
    }

    // More conditions...
}
```

Now imagine repeating this in every service.

Instead, we create our own application-level exceptions.
Exception translation
PostgreSQL error
SQLState 23505

SQLException


Custom Java exception
DuplicateEmployeeException

EmployeeService
Understands business meaning instead of SQL details

This process is called exception translation.

The database error:
SQLState 23505


Becomes an application error:
DuplicateEmployeeException


The service doesn't need to know PostgreSQL-specific codes.

## 7. Create an exception package
Our project structure:

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
├── exception/
│   ├── DatabaseException.java
│   ├── DuplicateEmployeeException.java
│   ├── EmployeeNotFoundException.java
│   └── DataIntegrityException.java
│
└── Main.java
```


## 8. Create DatabaseException
This is our generic database exception.

```
package com.example.jdbc.exception;

public class DatabaseException extends RuntimeException {

    public DatabaseException(String message) {
        super(message);
    }

    public DatabaseException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

#### Why extend RuntimeException?

Because our DAO will translate low-level checked SQLException into a meaningful application exception.

Then our service does not need to propagate SQLException everywhere.

However, this is a design choice. Checked custom exceptions can also be appropriate. The important thing is to choose a consistent application policy.

## 9. Create DuplicateEmployeeException

```
package com.example.jdbc.exception;

public class DuplicateEmployeeException extends RuntimeException {

    public DuplicateEmployeeException(String message) {
        super(message);
    }

    public DuplicateEmployeeException(
            String message,
            Throwable cause
    ) {
        super(message, cause);
    }
}
```

This represents:
Employee with the same email already exists


Notice the difference:
SQLException


is a technical database exception.
DuplicateEmployeeException


is an application-level exception with meaningful context.

## 10. Create EmployeeNotFoundException

```
package com.example.jdbc.exception;

public class EmployeeNotFoundException extends RuntimeException {

    public EmployeeNotFoundException(String message) {
        super(message);
    }
}
```

This represents a business/application situation:
Employee with ID 100 does not exist


It isn't necessarily a database failure.

A query can execute successfully but return no row.

That distinction is very important.
Database error vs application condition
Database error
SQL execution failed
Application condition
SQL succeeded but no employee found


Database error:
SQLException
SQLState 23505


Application condition:
findById(100)
→ query successfully executed
→ zero rows returned
→ EmployeeNotFoundException




## 11. Translate SQL errors inside EmployeeDAO
Now let's create a better DAO.

The DAO knows about:
- SQL
- PostgreSQL
- JDBC
- SQLState
- Database constraints
Therefore, the DAO is a good layer for translating database-specific failures into application-level exceptions.

```
package com.example.jdbc.dao;

import com.example.jdbc.config.DatabaseConfig;
import com.example.jdbc.exception.DatabaseException;
import com.example.jdbc.exception.DuplicateEmployeeException;
import com.example.jdbc.model.Employee;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.SQLException;

public class EmployeeDAO {

    public void save(Employee employee) {

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

            statement.setString(1, employee.getName());
            statement.setString(2, employee.getEmail());
            statement.setBigDecimal(3, employee.getSalary());
            statement.setString(4, employee.getDepartment());

            statement.executeUpdate();

        } catch (SQLException ex) {

            handleSQLException(ex);
        }
    }

    private void handleSQLException(SQLException ex) {

        String sqlState = ex.getSQLState();

        if ("23505".equals(sqlState)) {

            throw new DuplicateEmployeeException(
                "Employee email already exists",
                ex
            );
        }

        if ("23503".equals(sqlState)) {

            throw new DatabaseException(
                "Employee references invalid database data",
                ex
            );
        }

        throw new DatabaseException(
            "Failed to save employee",
            ex
        );
    }
}
```

Notice what happened.

The DAO has converted this:
SQLException
SQLState 23505


Into this:
DuplicateEmployeeException


And this:
SQLException
SQLState 23503


Into this:
DatabaseException


The original SQLException is still preserved as the cause.

This is important for debugging.

## 12. Exception propagation through DAO → Service → Main
Now let's understand the complete flow.


#### EmployeeDAO

```
public void save(Employee employee) {

    try {
        // Execute SQL

    } catch (SQLException ex) {

        if ("23505".equals(ex.getSQLState())) {
            throw new DuplicateEmployeeException(
                "Employee email already exists",
                ex
            );
        }

        throw new DatabaseException(
            "Database operation failed",
            ex
        );
    }
}
```

The DAO translates the technical error.

#### EmployeeService
The service handles business logic.

```
package com.example.jdbc.service;

import com.example.jdbc.dao.EmployeeDAO;
import com.example.jdbc.exception.DuplicateEmployeeException;
import com.example.jdbc.model.Employee;

public class EmployeeService {

    private final EmployeeDAO employeeDAO = new EmployeeDAO();

    public void createEmployee(Employee employee) {

        try {

            employeeDAO.save(employee);

            System.out.println("Employee created successfully");

        } catch (DuplicateEmployeeException ex) {

            System.out.println(
                "Business error: " + ex.getMessage()
            );

            throw ex;
        }
    }
}
```

Notice that the service doesn't check:
ex.getSQLState()


It doesn't need to know that PostgreSQL uses 23505.

It understands:
DuplicateEmployeeException


That is cleaner.

## Main.java

```
package com.example.jdbc;

import com.example.jdbc.exception.DatabaseException;
import com.example.jdbc.exception.DuplicateEmployeeException;
import com.example.jdbc.model.Employee;
import com.example.jdbc.service.EmployeeService;

import java.math.BigDecimal;

public class Main {

    public static void main(String[] args) {

        EmployeeService employeeService = new EmployeeService();

        Employee employee = new Employee(
            "Rahul",
            "rahul@example.com",
            new BigDecimal("50000"),
            "IT"
        );

        try {

            employeeService.createEmployee(employee);

        } catch (DuplicateEmployeeException ex) {

            System.out.println(
                "Cannot create employee: " + ex.getMessage()
            );

        } catch (DatabaseException ex) {

            System.out.println(
                "Database operation failed"
            );

            // Log the original cause for debugging
            ex.printStackTrace();
        }
    }
}
```

Architecture
Main
Application entry point / final handling

EmployeeService
Business logic

EmployeeDAO
SQL + JDBC + error translation

PostgreSQL
SQLState / database errors


## 13. Never swallow exceptions
A very common beginner mistake:

```
try {

    employeeDAO.save(employee);

} catch (SQLException ex) {

    System.out.println("Something went wrong");
}
```

The application now continues as though the operation was handled successfully.

Imagine:
```
createEmployee(employee);
sendWelcomeEmail(employee);
```

If the database insert fails but the exception is swallowed, you might still send the welcome email.

Now your application has inconsistent behavior.

Never silently ignore an exception unless you intentionally have a recovery or fallback strategy.
#### Bad
```
catch (SQLException ex) {
    // Ignore
}


Also bad
catch (SQLException ex) {
    System.out.println("Error");
}
```

The second one hides the actual cause.


#### Better

```
catch (SQLException ex) {

    throw new DatabaseException(
        "Failed to save employee",
        ex
    );
}
```

Or, if the exception is handled at the current layer:

```
catch (SQLException ex) {

    logger.error("Employee insert failed", ex);

    // Handle properly
}
```


## 14. Logging database exceptions
In a real Java application, use a logging framework instead of System.out.println().

For example, SLF4J.

```
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class EmployeeDAO {

    private static final Logger logger =
        LoggerFactory.getLogger(EmployeeDAO.class);
}


Then:
catch (SQLException ex) {

    logger.error(
        "Failed to insert employee with email {}",
        employee.getEmail(),
        ex
    );

    throw new DatabaseException(
        "Failed to insert employee",
        ex
    );
}
```

The final ex is important.

It logs the exception stack trace and cause.

What should you log?

Usually:
- Operation name
- Entity or record identifier
- SQLState
- Exception
- Request/job correlation ID if available
- Execution context that helps debugging
Avoid logging:
- Database passwords
- Authentication tokens
- Sensitive personal data unnecessarily
- Full SQL parameters containing confidential information

## 15. Batch exceptions — BatchUpdateException
When you execute:
statement.executeBatch();


If one or more operations fail, JDBC may throw:
BatchUpdateException


This is a subclass of:
SQLException


Example:

```
try {

    statement.executeBatch();

} catch (BatchUpdateException ex) {

    System.out.println("Batch failed");

    int[] updateCounts = ex.getUpdateCounts();

    for (int i = 0; i < updateCounts.length; i++) {

        System.out.println(
            "Batch item " + i +
            " result: " + updateCounts[i]
        );
    }

    throw ex;
}
```

Why is this important?

```
Suppose you insert 100 employees.
Employee 1 → SUCCESS
Employee 2 → SUCCESS
Employee 3 → SUCCESS
Employee 4 → FAILED
Employee 5 → ?
Employee 6 → ?
...
```

When a batch fails, the driver may report update counts for operations executed before the failure, and behavior can vary by driver and batch execution strategy.

You should not blindly assume:

100 operations
- 100 results
- every result always tells the exact outcome


The JDBC constants are important:

- Statement.SUCCESS_NO_INFO
- Statement.EXECUTE_FAILED


Example handling:

```
catch (BatchUpdateException ex) {

    int[] counts = ex.getUpdateCounts();

    for (int count : counts) {

        if (count == Statement.EXECUTE_FAILED) {
            System.out.println("Operation failed");
        } else if (count == Statement.SUCCESS_NO_INFO) {
            System.out.println("Operation succeeded; count unknown");
        } else {
            System.out.println("Rows affected: " + count);
        }
    }
}
```


## 16. Transaction failure and rollback
Let's combine our previous lessons.

Suppose:

```
BEGIN

Insert Employee 1
Insert Employee 2
Insert Employee 3
Insert Employee 4 → duplicate email
```

The batch fails.

What should happen?

If the whole import is intended to be atomic:

```
BEGIN
   Employee 1
   Employee 2
   Employee 3
   Employee 4 → ERROR
ROLLBACK
```

The DAO should not only throw an exception. It must also ensure that the transaction is rolled back when it owns the transaction.

#### Example

```
public void insertEmployees(
        List<Employee> employees
) throws SQLException {

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

        connection.setAutoCommit(false);

        try {

            for (Employee employee : employees) {

                statement.setString(1, employee.getName());
                statement.setString(2, employee.getEmail());
                statement.setBigDecimal(3, employee.getSalary());
                statement.setString(4, employee.getDepartment());

                statement.addBatch();
            }

            statement.executeBatch();

            connection.commit();

        } catch (SQLException ex) {

            try {
                connection.rollback();
            } catch (SQLException rollbackEx) {
                ex.addSuppressed(rollbackEx);
            }

            throw ex;

        } finally {

            try {
                connection.setAutoCommit(true);
            } catch (SQLException resetEx) {
                // Connection cleanup/reset failure.
            }
        }
    }
}
```

#### Why addSuppressed()?
Suppose:
1. The batch fails.
2. Rollback also fails.
We want to preserve both failures.

```
ex.addSuppressed(rollbackEx);
```

The original exception remains primary, and the rollback failure is retained as a suppressed exception.

In production, be careful with resetting auto-commit after a failed transaction. With a pooled connection, connection state must be reset correctly. Connection pools may reset some state when connections are returned, but you should not rely on that as a substitute for correct transaction handling.

## 17. Database error classification — how should your application react?
Not every database exception should be treated the same way.

The most important skill is to classify errors by their nature.
### Error classification

#### Permanent errors
Do not retry blindly

#### Transient errors
May be retryable


### A. Permanent errors
Usually retrying the same operation with the same data will not fix these.

|Error|	SQLState|	Action|
|-----|---------|---------|
|Duplicate email|	23505	|Correct data or handle conflict|
|Foreign key violation|	23503|	Correct relationship|
|NOT NULL violation|	23502|	Provide required data|
|Check constraint violation	|23514|	Correct input|
|Invalid SQL|	42601	|Fix query|
|Undefined table	|42P01|	Fix schema/query|
|Undefined column|	42703|	Fix schema/query|

#### Example:
```
Insert Rahul

Email = rahul@example.com

Email already exists

Retrying with same email
        ↓
Same error
```

Retrying is pointless.

### B. Transient errors
These may resolve if the operation is attempted again.

|Error	|SQLState	|Possible action|
|-------|-----------|---------------|
|Deadlock	|40P01	Retry transaction|
|Serialization failure	|40001	|Retry transaction|
|Connection interruption	|08xxx	|Reconnect if safe|
|Temporary database |unavailable	|Depends	Retry with backoff|
|Timeout	Depends	|Investigate/retry |selectively|

Example:
```
Transaction 1
     ↓
Deadlock detected
     ↓
PostgreSQL aborts transaction
     ↓
Retry the complete operation
     ↓
Success
```

#### Important retry rule
Do not retry only the failed SQL statement when the transaction itself has been aborted.

For example:
```
BEGIN
   UPDATE account A
   UPDATE account B
   DEADLOCK
```

The transaction is aborted.

Correct recovery:
```
ROLLBACK
    ↓
START NEW TRANSACTION
    ↓
Repeat entire logical operation
```


## 18. Retry strategy — Java implementation concept
Let's create a simple retry utility.

This is a learning implementation. Production retry logic should be more carefully designed around transaction boundaries, connection state, idempotency, retryable exceptions, and backoff.

```
package com.example.jdbc.util;

import java.sql.SQLException;

public class RetryExecutor {

    public static void execute(
            int maxAttempts,
            Runnable operation
    ) {

        for (int attempt = 1; attempt <= maxAttempts; attempt++) {

            try {

                operation.run();

                return;

            } catch (RuntimeException ex) {

                if (attempt == maxAttempts) {
                    throw ex;
                }

                try {

                    Thread.sleep(
                        100L * attempt
                    );

                } catch (InterruptedException interruptedEx) {

                    Thread.currentThread().interrupt();

                    throw new RuntimeException(
                        "Retry interrupted",
                        interruptedEx
                    );
                }
            }
        }
    }
}
```

But this generic version retries all runtime exceptions, which is not appropriate for production.

We should retry only known transient errors.

For example, classify the SQLState:
```
public static boolean isRetryable(SQLException ex) {

    String sqlState = ex.getSQLState();

    return "40001".equals(sqlState)
        || "40P01".equals(sqlState);
}
```

Do not retry every database error.

A retry strategy should generally include:
- Maximum retry count
- Backoff
- Jitter
- Retryable error classification
- Transaction restart
- Idempotency strategy
- Logging
- Cancellation/interrupt handling

We'll study retry strategies more deeply when we cover resilience and production database engineering.

## 19. Connection failures — what should Java do?
Suppose PostgreSQL is stopped.

Your application executes:
```
Connection connection = dataSource.getConnection();
```

It may fail.

Possible causes include:
- PostgreSQL is down.
- Host unreachable.
- Wrong port.
- Connection refused.
- Network failure.
- Invalid credentials.
- Connection timeout.
- Connection pool exhausted.

#### Example handling:

```
try {

    Connection connection = dataSource.getConnection();

} catch (SQLException ex) {

    if (ex.getSQLState() != null
        && ex.getSQLState().startsWith("08")) {

        System.out.println(
            "Database connection problem"
        );

    } else {

        System.out.println(
            "Database operation failed"
        );
    }
}
```

SQLState class 08 represents connection-related errors.

But don't automatically assume every 08xxx condition is recoverable in the same way. You need to inspect the operation, the connection state, and the failure.

## 20. Timeout handling
Database operations should not necessarily run forever.

JDBC supports statement/query timeout configuration:

```
PreparedStatement statement =
    connection.prepareStatement(sql);

statement.setQueryTimeout(10);
```

This sets the timeout in seconds, where supported by the driver.

If the query exceeds the timeout, JDBC may cancel the statement and report an exception.

You can also configure PostgreSQL:
```
SET statement_timeout = '10s';
```

For a particular session, or configure an appropriate timeout at a suitable level.

Three different timeout concepts
- Connection timeout → how long to wait to obtain a connection from the pool.
- Query timeout → how long a query is allowed to execute.
- Network/socket timeout → how long network communication may wait.
They solve different problems.

You already learned about HikariCP's:

connectionTimeout=30000


That is not the same as a SQL query timeout.

## 21. The exception hierarchy for our Employee project
Here is a simple architecture we will use.

```
Throwable
│
├── Error
│
└── Exception
    │
    ├── SQLException
    │   ├── BatchUpdateException
    │   └── SQLTransientException
    │
    └── RuntimeException
        │
        ├── DatabaseException
        │
        ├── DuplicateEmployeeException
        │
        ├── EmployeeNotFoundException
        │
        └── DataIntegrityException
```

We don't need to create a custom exception for every possible database error.

A practical initial hierarchy could be:
```
exception/
│
├── DatabaseException
├── DuplicateEmployeeException
├── EmployeeNotFoundException
└── DataIntegrityException
```

Later, in Spring Boot, you'll learn how Spring already provides a rich hierarchy of data-access exceptions, including:
- DataAccessException
- DuplicateKeyException
- DataIntegrityViolationException
- DataAccessResourceFailureException
- QueryTimeoutException
- TransientDataAccessException
That is one of the things Spring JDBC solves for you.

## 22. Recommended exception-handling architecture for Java JDBC
Your Employee application

```
Main.java
Application entry point

EmployeeService.java
Business logic and application rules

EmployeeDAO.java
JDBC + SQL + exception translation

PostgreSQL
Constraints + database errors + SQLState
```

#### Responsibilities

|Layer	|Responsibility|
|-------|--------------|
|PostgreSQL	|Enforce constraints, execute SQL, detect database errors|
|JDBC	|Expose database errors through SQLException|
|DAO	|Translate technical SQL errors into application exceptions|
|Service	|Handle business rules and coordinate operations|
|Main	|Handle application-level failures and output/logging|

This structure is the foundation you'll later use in Spring Boot.

## 23. What you should remember for interviews

### JDBC Exception Handling — Revision

#### SQLException
JDBC database error.

#### SQLState
Standardized error classification code.

#### Error code
Database/driver-specific numeric code.

#### Exception propagation
Exception moves upward through the method call stack.

#### Exception translation
Convert technical exceptions into meaningful application exceptions.

#### BatchUpdateException
Exception associated with failed JDBC batch execution.

#### Rollback
Undo transaction changes after failure.

#### Retry
Retry transient failures only, preferably the entire transaction.

#### Logging
Preserve the original exception and cause for debugging.

#### Never
Swallow exceptions or blindly retry every database failure.


#### Next topic: Phase 7.9 — JDBC Data Mapping & ResultSet to Java Objects

In the next lesson, we'll focus on one of the most important practical JDBC skills: converting database rows into Java objects properly, including NULL, BigDecimal, dates, timestamps, UUIDs, enums, JSON/JSONB, and creating reusable mapping patterns instead of writing repetitive code everywhere.