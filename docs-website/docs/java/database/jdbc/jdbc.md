# JDBC + SQL Deep Revision

## 7.1 JDBC — Java Database Connectivity

### 1. What is JDBC?
JDBC (Java Database Connectivity) is a Java API that allows Java applications to communicate with relational databases.

JDBC provides a standard API, while the database-specific driver handles communication with the database.

#### Why do we need JDBC?
Java doesn't inherently know how to communicate with PostgreSQL or MySQL.

We need a driver that translates Java JDBC operations into database-specific communication.

### 2. JDBC Architecture

```
┌─────────────────────────────┐
│       Java Application     │
│                             │
│ Controller / Service / DAO  │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│          JDBC API           │
│                             │
│ Connection                  │
│ PreparedStatement           │
│ ResultSet                  │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│       JDBC Driver           │
│                             │
│ PostgreSQL JDBC Driver      │
└──────────────┬──────────────┘
               │
               ▼
┌─────────────────────────────┐
│       PostgreSQL DB         │
└─────────────────────────────┘
```

#### Main JDBC components

- Driver - Communicates with the database
- Connection- 	Represents a database connection
- Statement-	Executes SQL
- PreparedStatement-	Executes parameterized SQL
- CallableStatement-	Calls stored procedures
- ResultSet-	Contains query results
- DriverManager-	Creates connections
- DataSource-	Provides database connections, often through a pool


## 7.1.1 JDBC Driver
A JDBC driver is a library that allows Java to communicate with a particular database.

For PostgreSQL:
Maven dependency
```
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.7.7</version>
</dependency>
```

For MySQL, you would use the MySQL Connector/J driver instead.

#### What happens when we add the dependency?
The driver provides the implementation for JDBC interfaces such as:
- Connection
- Statement
- PreparedStatement
- ResultSet


## 7.1.2 Connection
A Connection represents an active logical connection between your Java application and a database.

```
Connection connection =
    DriverManager.getConnection(
        "jdbc:postgresql://localhost:5432/mydb",
        "postgres",
        "password"
    );
```

Connection URL
```
jdbc:postgresql://localhost:5432/mydb
```

Breakdown:

- jdbc	- JDBC protocol
- postgresql - Database driver
- localhost -	Database host
- 5432 -	Database port
- mydb -	Database name

Important Connection methods
```
connection.close();
connection.commit();
connection.rollback();
connection.setAutoCommit(false);
connection.setAutoCommit(true);
connection.isClosed();
```


## 7.1.3 Statement
Statement is used to execute SQL queries.

```
Statement statement = connection.createStatement();

ResultSet resultSet = statement.executeQuery(
    "SELECT * FROM employees"
);
```

Problem with Statement
String username = request.getUsername();

String sql =
    "SELECT * FROM users WHERE username = '" + username + "'";


This is vulnerable to SQL Injection.

For example, a malicious username could contain SQL syntax that changes the query's behavior.

Therefore, in application code, prefer PreparedStatement.

## 7.1.4 PreparedStatement
A PreparedStatement allows you to write SQL with placeholders.

```
String sql = "SELECT * FROM employees WHERE id = ?";

PreparedStatement statement =
    connection.prepareStatement(sql);

statement.setInt(1, 10);

ResultSet resultSet = statement.executeQuery();


The ? is a parameter placeholder.
statement.setInt(1, 10);
statement.setString(2, "Supriyo");
statement.setDouble(3, 50000.0);
statement.setBoolean(4, true);
```

Why PreparedStatement?
1. Prevents SQL injection when used properly.
2. Separates SQL structure from input values.
3. Provides type-safe parameter binding.
4. Can allow the database to reuse execution plans.
5. Makes code cleaner.

answer
```
PreparedStatement is a parameterized SQL statement that allows values to be bound separately from SQL. It helps prevent SQL injection and provides better handling of SQL parameters.
```

## 7.1.5 CallableStatement
Used to call database stored procedures.

Suppose PostgreSQL has a function:

```
CREATE FUNCTION get_employee_count()
RETURNS INTEGER
LANGUAGE SQL
AS $$
    SELECT COUNT(*) FROM employees;
$$;
```

You could execute database functions using JDBC.
```
CallableStatement statement =
    connection.prepareCall("{? = call get_employee_count()}");

statement.registerOutParameter(1, Types.INTEGER);

statement.execute();

int count = statement.getInt(1);
```

In PostgreSQL, functions are commonly invoked using SQL queries too, but CallableStatement is part of the JDBC API for callable database routines.

## 7.1.6 ResultSet
ResultSet contains the rows returned by a query.

```
ResultSet resultSet = statement.executeQuery();


It uses a cursor.

Initially, the cursor is positioned before the first row.
while (resultSet.next()) {
    int id = resultSet.getInt("id");

    String name = resultSet.getString("name");

    System.out.println(id + " " + name);
}
```

#### ResultSet methods
```
resultSet.next();
resultSet.getInt("id");
resultSet.getString("name");
resultSet.getDouble("salary");
resultSet.getBoolean("active");
resultSet.getTimestamp("created_at");
```

Important
resultSet.next()


Moves the cursor to the next row.

It returns:
- true → a row exists
- false → no more rows

## 7.1.7 executeQuery(), executeUpdate(), execute()
Method	Use case	Return
executeQuery()	SELECT	ResultSet
executeUpdate()	INSERT, UPDATE, DELETE	Number of affected rows
execute()	General SQL execution	Boolean

#### SELECT
ResultSet resultSet =
    preparedStatement.executeQuery();


#### INSERT
int rows =
    preparedStatement.executeUpdate();

System.out.println("Inserted rows: " + rows);


#### UPDATE
int rows =
    preparedStatement.executeUpdate();


#### DELETE
int rows =
    preparedStatement.executeUpdate();


#### execute()
boolean result = preparedStatement.execute();


Useful when executing SQL where the result type might vary, such as multiple statements or database-specific operations.

## 7.1.8 Try-with-resources
Database resources must be closed.

#### The traditional approach:
```
Connection connection = null;

try {
    connection = DriverManager.getConnection(url, username, password);

    // Execute SQL

} finally {
    if (connection != null) {
        connection.close();
    }
}
```

Problem:
- Verbose
- Easy to forget closing resources
- Exception handling becomes complicated

#### Modern approach

```
try (
    Connection connection =
        DriverManager.getConnection(url, username, password);

    PreparedStatement statement =
        connection.prepareStatement(
            "SELECT * FROM employees WHERE id = ?"
        )
) {

    statement.setInt(1, 10);

    try (ResultSet resultSet = statement.executeQuery()) {

        if (resultSet.next()) {
            System.out.println(
                resultSet.getString("name")
            );
        }
    }

} catch (SQLException e) {
    e.printStackTrace();
}
```

Java automatically closes:
1. ResultSet
2. PreparedStatement
3. Connection
Resources are closed in reverse order of creation.

## 7.1.9 Practical JDBC Project Structure
Let's create a small standalone Java backend project to understand JDBC before using Spring Boot abstractions.

Suppose we are creating an Employee API with PostgreSQL.
#### Directory structure

```
jdbc-demo/
│
├── src/
│   └── main/
│       └── java/
│           └── com/
│               └── example/
│                   └── jdbc/
│                       │
│                       ├── Main.java
│                       │
│                       ├── config/
│                       │   └── DatabaseConfig.java
│                       │
│                       ├── model/
│                       │   └── Employee.java
│                       │
│                       ├── dao/
│                       │   └── EmployeeDAO.java
│                       │
│                       ├── service/
│                       │   └── EmployeeService.java
│                       │
│                       └── util/
│                           └── DatabaseConnection.java
│
└── pom.xml
```

Responsibility of each directory

```
Directory	Responsibility
config	Database configuration
model	Java objects representing data
dao	Database queries and persistence
service	Business logic
util	Reusable infrastructure utilities
Main	Application entry point
```
In a production application, the configuration and connection management would usually be combined with a DataSource rather than a manually implemented connection utility.

#### Step 1 — Create the database
```
CREATE DATABASE jdbc_demo;


Connect to the database and create a table.
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(150) UNIQUE,
    salary NUMERIC(10, 2)
);
```


#### Step 2 — Database configuration
config/DatabaseConfig.java

```
package com.example.jdbc.config;

public class DatabaseConfig {

    public static final String URL =
        "jdbc:postgresql://localhost:5432/jdbc_demo";

    public static final String USERNAME =
        "postgres";

    public static final String PASSWORD =
        "password";
}
```

For a learning project, this is okay.

For production, never hardcode passwords. Use environment variables, secrets management, or configuration external to your source code.

#### Step 3 — Connection utility
util/DatabaseConnection.java

```
package com.example.jdbc.util;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

import com.example.jdbc.config.DatabaseConfig;

public class DatabaseConnection {

    public static Connection getConnection()
            throws SQLException {

        return DriverManager.getConnection(
            DatabaseConfig.URL,
            DatabaseConfig.USERNAME,
            DatabaseConfig.PASSWORD
        );
    }
}
```

Now any class can request a database connection:
Connection connection =
    DatabaseConnection.getConnection();


This centralizes connection creation.

However, this implementation creates a new physical database connection every time you call getConnection().

That becomes a problem in backend applications.

#### Step 4 — Model
model/Employee.java

```
package com.example.jdbc.model;

public class Employee {

    private int id;
    private String name;
    private String email;
    private double salary;

    public Employee() {}

    public Employee(
            int id,
            String name,
            String email,
            double salary
    ) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.salary = salary;
    }

    public Employee(
            String name,
            String email,
            double salary
    ) {
        this.name = name;
        this.email = email;
        this.salary = salary;
    }

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }

    public String getEmail() {
        return email;
    }

    public double getSalary() {
        return salary;
    }

    public void setId(int id) {
        this.id = id;
    }

    public void setName(String name) {
        this.name = name;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public void setSalary(double salary) {
        this.salary = salary;
    }
}
```


#### Step 5 — DAO (Data Access Object)
DAO separates database operations from business logic.

dao/EmployeeDAO.java

```
package com.example.jdbc.dao;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;

import com.example.jdbc.model.Employee;
import com.example.jdbc.util.DatabaseConnection;

public class EmployeeDAO {

    public void save(Employee employee)
            throws SQLException {

        String sql = """
            INSERT INTO employees(name, email, salary)
            VALUES (?, ?, ?)
            """;

        try (
            Connection connection =
                DatabaseConnection.getConnection();

            PreparedStatement statement =
                connection.prepareStatement(sql)
        ) {

            statement.setString(1, employee.getName());
            statement.setString(2, employee.getEmail());
            statement.setDouble(3, employee.getSalary());

            statement.executeUpdate();
        }
    }

    public Employee findById(int id)
            throws SQLException {

        String sql =
            "SELECT * FROM employees WHERE id = ?";

        try (
            Connection connection =
                DatabaseConnection.getConnection();

            PreparedStatement statement =
                connection.prepareStatement(sql)
        ) {

            statement.setInt(1, id);

            try (ResultSet resultSet =
                    statement.executeQuery()) {

                if (resultSet.next()) {

                    return new Employee(
                        resultSet.getInt("id"),
                        resultSet.getString("name"),
                        resultSet.getString("email"),
                        resultSet.getDouble("salary")
                    );
                }
            }
        }

        return null;
    }

    public List<Employee> findAll()
            throws SQLException {

        List<Employee> employees = new ArrayList<>();

        String sql = "SELECT * FROM employees";

        try (
            Connection connection =
                DatabaseConnection.getConnection();

            PreparedStatement statement =
                connection.prepareStatement(sql);

            ResultSet resultSet =
                statement.executeQuery()
        ) {

            while (resultSet.next()) {

                Employee employee = new Employee(
                    resultSet.getInt("id"),
                    resultSet.getString("name"),
                    resultSet.getString("email"),
                    resultSet.getDouble("salary")
                );

                employees.add(employee);
            }
        }

        return employees;
    }

    public boolean deleteById(int id)
            throws SQLException {

        String sql =
            "DELETE FROM employees WHERE id = ?";

        try (
            Connection connection =
                DatabaseConnection.getConnection();

            PreparedStatement statement =
                connection.prepareStatement(sql)
        ) {

            statement.setInt(1, id);

            int rows =
                statement.executeUpdate();

            return rows > 0;
        }
    }
}
```

DAO responsibilities
```
EmployeeDAO
    |
    ├── save()
    ├── findById()
    ├── findAll()
    ├── update()
    └── deleteById()
```

DAO should generally contain:
- SQL queries
- JDBC operations
- Mapping database rows to Java objects
- Persistence-related logic

DAO should generally not contain:
- Business rules
- HTTP request handling
- Controller logic
- Authentication logic

#### Step 6 — Service Layer
service/EmployeeService.java
```
package com.example.jdbc.service;

import java.sql.SQLException;
import java.util.List;

import com.example.jdbc.dao.EmployeeDAO;
import com.example.jdbc.model.Employee;

public class EmployeeService {

    private final EmployeeDAO employeeDAO;

    public EmployeeService() {
        this.employeeDAO = new EmployeeDAO();
    }

    public void createEmployee(Employee employee)
            throws SQLException {

        // Business validation

        if (employee.getSalary() <= 0) {
            throw new IllegalArgumentException(
                "Salary must be greater than zero"
            );
        }

        employeeDAO.save(employee);
    }

    public Employee getEmployee(int id)
            throws SQLException {

        return employeeDAO.findById(id);
    }

    public List<Employee> getAllEmployees()
            throws SQLException {

        return employeeDAO.findAll();
    }
}
```


## 7.2 Connection Pooling
This is one of the most important backend database concepts.
Without connection pooling
Imagine 100 API requests arrive.

Each request:
```
Request
  |
  ▼
Create new DB connection
  |
  ▼
Execute SQL
  |
  ▼
Close DB connection
```

Creating a database connection is expensive because it involves:
- Network communication
- Authentication
- TCP connection establishment
- Database session creation
- Resource allocation
If every request creates a new connection, your application wastes resources.

With connection pooling
A connection pool maintains a collection of reusable database connections.
```
                    ┌─────────────────┐
                    │  Java Backend   │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │ Connection Pool │
                    └────────┬────────┘
                             │
              ┌──────────────┼──────────────┐
              ▼              ▼              ▼
         Connection 1   Connection 2   Connection 3
              │              │              │
              └──────────────┼──────────────┘
                             ▼
                        PostgreSQL

```
Instead of creating a new physical connection, your application borrows one from the pool.

When you call:
connection.close();


with a pooled connection, it usually doesn't destroy the physical database connection.

Instead:
```
Application
    |
    | connection.close()
    ▼
Connection Pool
    |
    | Return connection to pool
    ▼
Available for another request
```

This is an extremely important distinction.

## 7.2.1 HikariCP
HikariCP is a high-performance JDBC connection pool.

It is widely used in Spring Boot applications.

Its responsibilities include:
- Creating connections
- Maintaining connections
- Reusing connections
- Managing pool size
- Removing unhealthy connections
- Handling connection timeout
- Managing idle connections

HikariCP architecture

```
Request 1 ─────┐
Request 2 ─────┤
Request 3 ─────┤
Request 4 ─────┤
Request 5 ─────┤
               ▼
        ┌───────────────┐
        │ HikariCP Pool │
        └───────┬───────┘
                │
       ┌────────┼────────┐
       ▼        ▼        ▼
     Conn 1   Conn 2   Conn 3
       │        │        │
       └────────┼────────┘
                ▼
           PostgreSQL

```

## 7.2.2 Pool Size
Suppose:
Maximum pool size = 10


Your application can have up to 10 connections available through that pool.

If 10 connections are currently being used and another request needs a connection:
```
Request 11
    |
    ▼
Pool exhausted
    |
    ▼
Wait for available connection
```

If no connection becomes available before the timeout:
Connection timeout exception


Important
More connections do not always mean better performance.

Your database also has a maximum connection capacity.

For example:
Application 1: 20 connections
Application 2: 20 connections
Application 3: 20 connections


Your database now potentially receives 60 connections.

Too many connections can cause:
- Memory pressure
- Context switching
- Database performance degradation
- Increased resource consumption

## 7.2.3 HikariCP Configuration
In a standalone JDBC application, you can use HikariCP directly.
#### Maven dependency

```
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>6.3.0</version>
</dependency>
```

#### Directory structure
```
jdbc-demo/
│
├── config/
│   └── DatabaseConfig.java
│
├── util/
│   └── DatabaseConnection.java
│
├── dao/
│   └── EmployeeDAO.java
│
└── service/
    └── EmployeeService.java
```

#### DatabaseConfig.java
```
package com.example.jdbc.config;

import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;

public class DatabaseConfig {

    private static final HikariDataSource dataSource;

    static {

        HikariConfig config = new HikariConfig();

        config.setJdbcUrl(
            "jdbc:postgresql://localhost:5432/jdbc_demo"
        );

        config.setUsername("postgres");
        config.setPassword("password");

        config.setMaximumPoolSize(10);
        config.setMinimumIdle(2);

        config.setConnectionTimeout(30000);
        config.setIdleTimeout(600000);
        config.setMaxLifetime(1800000);

        dataSource = new HikariDataSource(config);
    }

    public static HikariDataSource getDataSource() {
        return dataSource;
    }
}
```

#### Connection utility

```
package com.example.jdbc.util;

import java.sql.Connection;
import java.sql.SQLException;

import com.example.jdbc.config.DatabaseConfig;

public class DatabaseConnection {

    public static Connection getConnection()
            throws SQLException {

        return DatabaseConfig
            .getDataSource()
            .getConnection();
    }
}
```

Now your DAO doesn't need to know whether the connection comes from:
- DriverManager
- HikariCP
- Another connection pool
It simply requests a connection.

## 7.2.4 HikariCP Configuration — Interview Revision
```
Property	Meaning	Example
maximumPoolSize	- Maximum number of connections	10
minimumIdle -	Minimum idle connections maintained	2
connectionTimeout -	Maximum time to wait for a connection	30,000 ms
idleTimeout -	How long an idle connection can remain	600,000 ms
maxLifetime -	Maximum lifetime of a connection	1,800,000 ms
```
Connection timeout
```
config.setConnectionTimeout(30000);
```

If no connection is available within 30 seconds, HikariCP throws an exception.

#### Idle timeout
config.setIdleTimeout(600000);

An idle connection may be removed from the pool after 10 minutes, provided the pool has more than its minimum idle connections.

#### Maximum lifetime
config.setMaxLifetime(1800000);

A connection is retired after 30 minutes.

This helps prevent connections from becoming stale due to network infrastructure or database-side connection limits.

#### What does Connection.close() mean with HikariCP?
This is a common interview question.

```
try (Connection connection =
        dataSource.getConnection()) {

    // Execute SQL

}
```

When the code finishes:
connection.close();


The connection is returned to the pool, not necessarily physically closed.

This is why try-with-resources is still necessary when using a connection pool.

## 7.2.5 JDBC vs Connection Pool
JDBC -	Connection Pool
API for database communication -	Manages reusable connections
Provides Connection, Statement, ResultSet -	Provides connections efficiently
Executes SQL -	Reduces connection creation overhead
Standard Java API	- Implementation such as HikariCP

```
JDBC
  └── Defines database communication API

HikariCP
  └── Manages connection lifecycle and reuse

PostgreSQL Driver
  └── Implements communication with PostgreSQL
```


How JDBC Is Actually Used in Spring Boot
In a real Spring Boot project, you normally don't manually create a HikariCP configuration class or DriverManager connection utility.

Spring Boot automatically configures:
```
Spring Boot
    |
    ▼
DataSource
    |
    ▼
HikariCP
    |
    ▼
PostgreSQL JDBC Driver
    |
    ▼
PostgreSQL
```

HikariCP is the default connection pool in typical Spring Boot JDBC setups when available.
#### Typical Spring Boot directory structure
For your backend projects, this is a structure I recommend learning:

```
helyx-data-api/
│
├── src/
│   └── main/
│       ├── java/
│       │   └── com/
│       │       └── helyx/
│       │           └── dataapi/
│       │
│       │               ├── DataApiApplication.java
│       │
│       │               ├── config/
│       │               │   └── DatabaseConfig.java
│       │
│       │               ├── controller/
│       │               │   └── EmployeeController.java
│       │
│       │               ├── service/
│       │               │   └── EmployeeService.java
│       │
│       │               ├── repository/
│       │               │   └── EmployeeRepository.java
│       │
│       │               ├── dto/
│       │               │   └── EmployeeDTO.java
│       │
│       │               ├── entity/
│       │               │   └── Employee.java
│       │
│       │               └── exception/
│       │                   └── GlobalExceptionHandler.java
│       │
│       └── resources/
│           ├── application.properties
│           └── application-dev.properties
│
└── pom.xml
```

Notice that Spring Boot projects generally use repository instead of dao.

The concepts are very similar:
```
DAO       → Data Access Object
Repository → Abstraction for data access
```


#### Spring Boot JDBC Dependencies


For a Spring Boot project using JDBC directly:
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```

PostgreSQL driver:

```
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
```

You normally don't need to manually add HikariCP because Spring Boot's JDBC starter brings in the necessary connection pooling dependency.

### Spring Boot Database Configuration
#### application.properties

```
spring.datasource.url=jdbc:postgresql://localhost:5432/jdbc_demo

spring.datasource.username=postgres

spring.datasource.password=${DB_PASSWORD}

spring.datasource.driver-class-name=org.postgresql.Driver


HikariCP properties:
spring.datasource.hikari.maximum-pool-size=10

spring.datasource.hikari.minimum-idle=2

spring.datasource.hikari.connection-timeout=30000

spring.datasource.hikari.idle-timeout=600000

spring.datasource.hikari.max-lifetime=1800000
```

Spring Boot automatically creates the DataSource.

You can inject it:

```
@Autowired
private DataSource dataSource;
```

Or preferably use constructor injection:
```
@Service
public class EmployeeService {

    private final DataSource dataSource;

    public EmployeeService(DataSource dataSource) {
        this.dataSource = dataSource;
    }
}
```

You normally don't need to manually instantiate:
new HikariDataSource();


in a Spring Boot application.

## 7.3 Transactions
A transaction is a group of database operations treated as one logical unit of work.

Suppose we transfer ₹1,000 from Account A to Account B.
```
Account A: ₹5,000
Account B: ₹2,000
```

We need:
1. Deduct ₹1,000 from Account A
2. Add ₹1,000 to Account B


Expected result:
```
Account A: ₹4,000
Account B: ₹3,000
```

But suppose the first query succeeds and the second query fails.

Without a transaction:
```
Account A: ₹4,000
Account B: ₹2,000
```

Money is lost from the logical transfer.

With a transaction:
```
BEGIN TRANSACTION

Deduct ₹1,000 from A

Add ₹1,000 to B

If everything succeeds:
    COMMIT

If anything fails:
    ROLLBACK


Result:
Either:

A = ₹4,000
B = ₹3,000

OR:

A = ₹5,000
B = ₹2,000
```

Never an inconsistent intermediate result.

## 7.3.1 Auto-commit
By default, JDBC connections usually operate in auto-commit mode.

```
connection.getAutoCommit();

Typically:
true
```

When auto-commit is enabled:

```
statement.executeUpdate(
    "UPDATE accounts SET balance = balance - 1000 WHERE id = 1"
);
```

The operation is committed automatically when it completes successfully.

Conceptually:
```
SQL Operation
     |
     ▼
Execute
     |
     ▼
Automatically COMMIT
```


## 7.3.2 Manual Transaction
Disable auto-commit:

```
connection.setAutoCommit(false);
```

Now you control when to commit or rollback.
```
connection.commit();
connection.rollback();
```

Example
```
Connection connection = null;

try {

    connection =
        DatabaseConnection.getConnection();

    connection.setAutoCommit(false);

    // Operation 1
    PreparedStatement debit =
        connection.prepareStatement(
            """
            UPDATE accounts
            SET balance = balance - ?
            WHERE id = ?
            """
        );

    debit.setDouble(1, 1000);
    debit.setInt(2, 1);

    debit.executeUpdate();

    // Operation 2
    PreparedStatement credit =
        connection.prepareStatement(
            """
            UPDATE accounts
            SET balance = balance + ?
            WHERE id = ?
            """
        );

    credit.setDouble(1, 1000);
    credit.setInt(2, 2);

    credit.executeUpdate();

    // Both operations successful
    connection.commit();

} catch (SQLException e) {

    if (connection != null) {
        connection.rollback();
    }

    throw e;

} finally {

    if (connection != null) {
        connection.close();
    }
}
```

Transaction lifecycle
```
Connection
    |
    ▼
setAutoCommit(false)
    |
    ▼
SQL Operation 1
    |
    ▼
SQL Operation 2
    |
    ├── Success ──► commit()
    │
    └── Failure ──► rollback()
```


## 7.3.3 Transaction Boundaries
A transaction boundary determines:
Where a transaction starts and where it ends.

For example:
```
Transfer money
    |
    ├── Validate account
    ├── Debit account
    ├── Credit account
    └── Save transfer history
```

All these database operations may belong to one transaction.

A transaction should generally be designed around a single logical business operation.

Bad:
```
Transaction starts
    |
    ▼
10 unrelated operations
    |
    ▼
Transaction ends
```

Long-running transactions can cause:
- Locks to remain active
- Increased contention
- Deadlocks
- Reduced concurrency
- Database resource consumption

## 7.3.4 ACID Properties
ACID describes the properties that make database transactions reliable.

```
Property	Meaning	Example
Atomicity	All operations succeed or all fail	Debit + credit
Consistency	Database remains valid	Balance cannot violate constraints
Isolation	Concurrent transactions don't improperly interfere	Two users updating account
Durability	Committed changes survive failure	Committed transfer remains saved
```
Atomicity
Operation A + Operation B

Both succeed
OR
Both rollback


Consistency
A transaction moves the database from one valid state to another valid state.
Isolation
Concurrent transactions behave according to the configured isolation level.
Durability
Once committed, changes should persist even if the application crashes.

## 7.3.5 Transactions in Spring Boot
In Spring Boot, you usually don't manually manage:
```
connection.setAutoCommit(false);

connection.commit();

connection.rollback();
```

Instead, Spring provides declarative transactions.

```
@Transactional
@Service
public class AccountService {

    private final AccountRepository accountRepository;

    public AccountService(
            AccountRepository accountRepository
    ) {
        this.accountRepository = accountRepository;
    }

    @Transactional
    public void transfer(
            int fromAccount,
            int toAccount,
            double amount
    ) {

        accountRepository.debit(
            fromAccount,
            amount
        );

        accountRepository.credit(
            toAccount,
            amount
        );
    }
}
```

Spring manages the transaction.

Conceptually:
Spring Transaction Manager
          |
          ▼
BEGIN TRANSACTION
          |
          ▼
accountRepository.debit()
          |
          ▼
accountRepository.credit()
          |
          ▼
     Method completes
          |
          ├── Success → COMMIT
          │
          └── Exception → ROLLBACK


Important default behavior
By default, Spring's declarative transaction management rolls back on:
- RuntimeException
- Error
It does not automatically roll back on every checked exception.

You can customize:
@Transactional(
    rollbackFor = Exception.class
)



JDBC vs Spring JDBC vs JPA
You should understand the difference because interviewers often ask about it.
Technology	What it does
JDBC	Low-level database API
Spring JDBC / JdbcTemplate	Simplifies JDBC operations
JPA	ORM specification
Hibernate	Common JPA implementation
Spring Data JPA	Simplifies repository-based JPA development

JDBC
You manually write:
Connection
PreparedStatement
ResultSet


Spring JdbcTemplate
Spring manages much of the repetitive JDBC infrastructure.

You still write SQL.
JPA/Hibernate
You work with entities and ORM abstractions.

For example:
employeeRepository.save(employee);


Hibernate generates or executes the database operations.

Practical JDBC in Spring Boot — JdbcTemplate
Since you're learning backend development with Spring Boot, JdbcTemplate is an important practical step between raw JDBC and JPA.

It helps you understand SQL and database integration without manually managing connections, statements, and result sets.
1. Repository
@Repository
public class EmployeeRepository {

    private final JdbcTemplate jdbcTemplate;

    public EmployeeRepository(
            JdbcTemplate jdbcTemplate
    ) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public int save(Employee employee) {

        String sql = """
            INSERT INTO employees(name, email, salary)
            VALUES (?, ?, ?)
            """;

        return jdbcTemplate.update(
            sql,
            employee.getName(),
            employee.getEmail(),
            employee.getSalary()
        );
    }
}


JdbcTemplate.update() is commonly used for:
- INSERT
- UPDATE
- DELETE
It returns the number of affected rows.

2. Query for one record
public Employee findById(int id) {

    String sql = """
        SELECT id, name, email, salary
        FROM employees
        WHERE id = ?
        """;

    return jdbcTemplate.queryForObject(
        sql,
        (rs, rowNum) -> new Employee(
            rs.getInt("id"),
            rs.getString("name"),
            rs.getString("email"),
            rs.getDouble("salary")
        ),
        id
    );
}


This lambda is a RowMapper.

Its responsibility:
Database Row
     |
     ▼
ResultSet
     |
     ▼
RowMapper
     |
     ▼
Java Object



3. Query for multiple records
public List<Employee> findAll() {

    String sql = """
        SELECT id, name, email, salary
        FROM employees
        """;

    return jdbcTemplate.query(
        sql,
        (rs, rowNum) -> new Employee(
            rs.getInt("id"),
            rs.getString("name"),
            rs.getString("email"),
            rs.getDouble("salary")
        )
    );
}



JDBC Revision Cheat Sheet
Connection

Establishes communication with the database.
Connection connection = dataSource.getConnection();



PreparedStatement

Executes parameterized SQL.
PreparedStatement ps =
    connection.prepareStatement(sql);



ResultSet

Reads query results.
while (rs.next()) {
    // Read row
}



executeQuery()

Used for SELECT.
ResultSet rs = ps.executeQuery();



executeUpdate()

Used for INSERT, UPDATE, DELETE.
int affectedRows = ps.executeUpdate();



execute()

General-purpose SQL execution.
boolean result = ps.execute();



Connection Pool

Reuses database connections.
HikariCP



Transaction

Controls a group of operations.
connection.commit();
connection.rollback();



Spring Transaction

Declarative transaction management.
@Transactional



Common Interview Questions
1. What is JDBC?
Java API used to connect and interact with relational databases.
2. Difference between Statement and PreparedStatement?
Statement executes SQL directly, whereas PreparedStatement supports parameterized SQL and helps prevent SQL injection.
3. What is ResultSet?
An object containing rows returned by a database query, accessed using a cursor.
4. Difference between executeQuery and executeUpdate?
- executeQuery() → returns ResultSet, typically for SELECT.
- executeUpdate() → returns affected row count, typically for INSERT, UPDATE, DELETE.
5. Why use try-with-resources?
Automatically closes resources implementing AutoCloseable, preventing resource leaks.
6. Why do we need connection pooling?
Creating database connections is expensive. Pooling allows connections to be reused.
7. What is HikariCP?
A high-performance JDBC connection pool that manages and reuses database connections.
8. What happens when a pooled connection is closed?
The logical connection is usually returned to the pool instead of physically closing the database connection.
9. What is auto-commit?
Automatically commits each successful SQL operation.
10. What is rollback?
Reverts uncommitted changes in the current transaction.
11. What is @Transactional?
A Spring annotation that enables declarative transaction management around a method or class.
12. What is the difference between JDBC and JPA?
JDBC gives you direct SQL execution and manual mapping. JPA provides ORM abstractions for mapping Java objects to database tables.

Your Revision Checklist
When you revise Phase 7, you should be able to recall these without reading the explanation:
- [ ] JDBC architecture
- [ ] JDBC Driver
- [ ] Connection
- [ ] Statement
- [ ] PreparedStatement
- [ ] SQL injection
- [ ] CallableStatement
- [ ] ResultSet and cursor
- [ ] executeQuery()
- [ ] executeUpdate()
- [ ] execute()
- [ ] Try-with-resources
- [ ] DAO pattern
- [ ] Connection pooling
- [ ] HikariCP
- [ ] Pool size
- [ ] Connection timeout
- [ ] Idle timeout
- [ ] Maximum lifetime
- [ ] Auto-commit
- [ ] Commit
- [ ] Rollback
- [ ] Transaction boundaries
- [ ] ACID
- [ ] Spring @Transactional

Recommended practical learning sequence
For your Spring Boot backend learning, I recommend practicing these in order:
1. Create a Java project with raw JDBC.
2. Connect to PostgreSQL using DriverManager.
3. Insert a record using PreparedStatement.
4. Read records using ResultSet.
5. Create a DAO layer.
6. Implement CRUD.
7. Add HikariCP.
8. Observe connection reuse.
9. Implement manual transactions.
10. Make a money-transfer transaction.
11. Build the same CRUD project in Spring Boot using JdbcTemplate.
12. Implement @Transactional.
13. Compare raw JDBC, JdbcTemplate, and Spring Data JPA.
The next topic in your revision should be Connection Pooling in depth, followed by Transactions and Isolation Levels. SQL Deep can remain a separate module as you planned.

For your notes, I can also turn this into a compact JDBC interview revision sheet with concept → syntax → practical example → interview trap for each topic.