# Phase 7.11 — JDBC Advanced Patterns & Architecture

### 7.11.1 — The Problem with Poor JDBC Architecture
Imagine you write your application like this:

```
public class Main {

    public static void main(String[] args) throws Exception {

        Connection connection =
                DriverManager.getConnection(
                        "jdbc:postgresql://localhost:5432/employee_db",
                        "postgres",
                        "password"
                );

        String sql =
                "SELECT * FROM employees WHERE id = ?";

        PreparedStatement ps =
                connection.prepareStatement(sql);

        ps.setInt(1, 101);

        ResultSet rs = ps.executeQuery();

        while (rs.next()) {

            String name = rs.getString("name");

            System.out.println(name);
        }

        // Business logic
        // Validation
        // Update database
        // Logging
        // Exception handling
        // Transaction handling
    }
}
```

This works.

But as your application grows, the problems become obvious.
Poor architecture problems

- SQL mixed with business logic
- Connection management mixed with application logic
- Duplicate SQL across multiple classes
- Difficult testing
- Difficult transaction management
- Difficult to change database
- Difficult to reuse logic
- Main class becomes huge
- Exception handling becomes inconsistent

For a small application, this might be acceptable temporarily.

For a real backend, we should separate responsibilities.

## 7.11.2 — The Layered JDBC Architecture

A clean Java JDBC application can use the following layers.

- Main / Application
Starts the application and coordinates execution

- Service Layer
Business logic, validation, workflow, transaction coordination

- DAO / Repository Layer
SQL execution and persistence operations

- Mapper Layer
ResultSet → Java objects

- Database Layer
Connection pool, JDBC, PostgreSQL


```
Main - Application startup
Service -	Business logic
DAO/Repository -	Database operations
Mapper -	Data conversion
Database - configuration	Connections and pool
Model -	Application data representation
```

## 7.11.3 — Understand the Responsibility of Each Layer
### 1. Model Layer
Represents application data.

```
public class Employee {

    private int id;
    private String name;
    private String email;
    private BigDecimal salary;

    // Constructors, getters and setters
}
```

The model should not normally contain JDBC code.

Avoid:
```
public class Employee {

    private Connection connection;

    public void save() {
        // SQL inside model
    }
}
```

This tightly couples your data model to the database infrastructure.

### 2. Mapper Layer
Responsible for converting:
```
ResultSet → Employee
```

Example:
```
public class EmployeeMapper {

    public static Employee map(ResultSet rs)
            throws SQLException {

        Employee employee = new Employee();

        employee.setId(rs.getInt("id"));
        employee.setName(rs.getString("name"));
        employee.setEmail(rs.getString("email"));
        employee.setSalary(rs.getBigDecimal("salary"));

        return employee;
    }
}
```

Mapper should not:
- Execute SQL
- Manage connections
- Decide business rules
- Commit transactions
It should map data.

### 3. DAO Layer
DAO stands for:
```
Data Access Object
```
Its responsibility is to interact with the database.

Example:
```
public class EmployeeDAO {

    public Employee findById(int id) {
        // SQL
        // PreparedStatement
        // ResultSet
        // Mapping
    }

    public List<Employee> findAll() {
        // SELECT
    }

    public int insert(Employee employee) {
        // INSERT
    }

    public int update(Employee employee) {
        // UPDATE
    }

    public int delete(int id) {
        // DELETE
    }
}
```

The DAO knows about:
- SQL
- JDBC
- Tables
- Database constraints
- Database operations
It should generally not decide whether a user is allowed to perform a particular business action.

### 4. Service Layer
The service layer contains business rules.

For example:
Employees cannot have salaries below the minimum allowed salary.
```
public class EmployeeService {

    private final EmployeeDAO employeeDAO;

    public EmployeeService(EmployeeDAO employeeDAO) {
        this.employeeDAO = employeeDAO;
    }

    public void createEmployee(Employee employee) {

        if (employee.getSalary() == null) {
            throw new IllegalArgumentException(
                    "Salary cannot be null"
            );
        }

        if (employee.getSalary().compareTo(
                new BigDecimal("15000")
        ) < 0) {

            throw new IllegalArgumentException(
                    "Salary cannot be below 15000"
            );
        }

        employeeDAO.insert(employee);
    }
}
```

The service should decide:
- Whether the operation is valid
- Whether multiple DAO calls are needed
- How business operations are coordinated
- Where transaction boundaries belong
The DAO should execute the actual database operation.

## 7.11.4 — DAO vs Repository
You will often hear these two terms.

#### DAO
A DAO is focused on data-access operations.
```
employeeDAO.findById(101);
employeeDAO.insert(employee);
employeeDAO.update(employee);
employeeDAO.delete(101);
```

It is generally closely related to the persistence mechanism.

#### Repository
A Repository is often modeled as a collection-like abstraction around domain objects.
```
employeeRepository.findById(101);
employeeRepository.save(employee);
employeeRepository.delete(employee);
```

It may hide more of the persistence implementation.

#### DAO
- Database-centric
- SQL-oriented
- CRUD operations
- JDBC-focused
- Explicit persistence logic


#### Repository
- Domain-oriented abstraction
- Collection-like interface
- Can hide persistence details
- Useful for application architecture



In pure JDBC, using a DAO is a very natural starting point.

## 7.11.5 — Dependency Injection Without Spring
You previously learned that Spring provides Dependency Injection.

But DI is not exclusive to Spring.

You can perform it manually.

Bad approach
```
public class EmployeeService {

    private EmployeeDAO employeeDAO =
            new EmployeeDAO();
}
```

Why is this problematic?

EmployeeService is tightly coupled to the concrete implementation.

It creates its own dependency.

That makes testing and replacement harder.

Better approach
```
public class EmployeeService {

    private final EmployeeDAO employeeDAO;

    public EmployeeService(EmployeeDAO employeeDAO) {
        this.employeeDAO = employeeDAO;
    }
}
```

Now the dependency is provided externally.

This is called constructor injection.


## 7.11.6 — Create an Interface for the DAO
Now we can take the architecture one step further.

Instead of depending directly on:
EmployeeDAO


we can depend on an interface.

### Step 1 — Create EmployeeRepository
```
package com.example.repository;

import java.sql.SQLException;
import java.util.List;
import java.util.Optional;

import com.example.model.Employee;

public interface EmployeeRepository {

    Optional<Employee> findById(int id) throws SQLException;

    List<Employee> findAll() throws SQLException;

    int insert(Employee employee) throws SQLException;

    int update(Employee employee) throws SQLException;

    int deleteById(int id) throws SQLException;
}
```

Notice that we use:
```
Optional<Employee>
```

instead of returning null for a missing employee.

This makes the absence of a result explicit.

### Step 2 — Implement the interface using JDBC
```
package com.example.dao;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

import com.example.mapper.EmployeeMapper;
import com.example.model.Employee;
import com.example.repository.EmployeeRepository;

public class JdbcEmployeeDAO implements EmployeeRepository {

    private final Connection connection;

    public JdbcEmployeeDAO(Connection connection) {
        this.connection = connection;
    }

    @Override
    public Optional<Employee> findById(int id)
            throws SQLException {

        String sql = """
                SELECT id, name, email, salary
                FROM employees
                WHERE id = ?
                """;

        try (PreparedStatement ps =
                connection.prepareStatement(sql)) {

            ps.setInt(1, id);

            try (ResultSet rs = ps.executeQuery()) {

                if (rs.next()) {
                    return Optional.of(
                            EmployeeMapper.map(rs)
                    );
                }

                return Optional.empty();
            }
        }
    }

    @Override
    public List<Employee> findAll()
            throws SQLException {

        List<Employee> employees = new ArrayList<>();

        String sql = """
                SELECT id, name, email, salary
                FROM employees
                ORDER BY id
                """;

        try (
            PreparedStatement ps =
                    connection.prepareStatement(sql);

            ResultSet rs = ps.executeQuery()
        ) {

            while (rs.next()) {

                employees.add(
                        EmployeeMapper.map(rs)
                );
            }
        }

        return employees;
    }

    @Override
    public int insert(Employee employee)
            throws SQLException {

        String sql = """
                INSERT INTO employees(name, email, salary)
                VALUES (?, ?, ?)
                """;

        try (PreparedStatement ps =
                connection.prepareStatement(sql)) {

            ps.setString(1, employee.getName());
            ps.setString(2, employee.getEmail());
            ps.setBigDecimal(3, employee.getSalary());

            return ps.executeUpdate();
        }
    }

    @Override
    public int update(Employee employee)
            throws SQLException {

        String sql = """
                UPDATE employees
                SET name = ?, email = ?, salary = ?
                WHERE id = ?
                """;

        try (PreparedStatement ps =
                connection.prepareStatement(sql)) {

            ps.setString(1, employee.getName());
            ps.setString(2, employee.getEmail());
            ps.setBigDecimal(3, employee.getSalary());
            ps.setInt(4, employee.getId());

            return ps.executeUpdate();
        }
    }

    @Override
    public int deleteById(int id)
            throws SQLException {

        String sql = """
                DELETE FROM employees
                WHERE id = ?
                """;

        try (PreparedStatement ps =
                connection.prepareStatement(sql)) {

            ps.setInt(1, id);

            return ps.executeUpdate();
        }
    }
}
```

Now your service depends on an abstraction:
EmployeeRepository


But the implementation is:
JdbcEmployeeDAO


This is a useful architectural pattern.
Service depends on interface
EmployeeRepository


Actual implementation
JdbcEmployeeDAO


Later, you could create another implementation without changing the service.
- InMemoryEmployeeRepository
- PostgreSQLEmployeeRepository


## 7.11.7 — Refactor EmployeeService
Now the service should depend on the repository interface, not the JDBC implementation.
```
package com.example.service;

import java.math.BigDecimal;
import java.sql.SQLException;
import java.util.List;
import java.util.Optional;

import com.example.model.Employee;
import com.example.repository.EmployeeRepository;

public class EmployeeService {

    private final EmployeeRepository employeeRepository;

    public EmployeeService(
            EmployeeRepository employeeRepository
    ) {
        this.employeeRepository = employeeRepository;
    }

    public Optional<Employee> getEmployeeById(int id)
            throws SQLException {

        if (id <= 0) {
            throw new IllegalArgumentException(
                    "Employee ID must be positive"
            );
        }

        return employeeRepository.findById(id);
    }

    public List<Employee> getAllEmployees()
            throws SQLException {

        return employeeRepository.findAll();
    }

    public void createEmployee(Employee employee)
            throws SQLException {

        validateEmployee(employee);

        employeeRepository.insert(employee);
    }

    private void validateEmployee(Employee employee) {

        if (employee == null) {
            throw new IllegalArgumentException(
                    "Employee cannot be null"
            );
        }

        if (employee.getName() == null ||
                employee.getName().isBlank()) {

            throw new IllegalArgumentException(
                    "Employee name is required"
            );
        }

        if (employee.getEmail() == null ||
                employee.getEmail().isBlank()) {

            throw new IllegalArgumentException(
                    "Employee email is required"
            );
        }

        if (employee.getSalary() == null) {

            throw new IllegalArgumentException(
                    "Employee salary is required"
            );
        }

        if (employee.getSalary().compareTo(
                BigDecimal.ZERO
        ) < 0) {

            throw new IllegalArgumentException(
                    "Salary cannot be negative"
            );
        }
    }
}
```

What changed?
Previously:
- EmployeeService → JdbcEmployeeDAO


Now:
- EmployeeService → EmployeeRepository


This reduces coupling.

## 7.11.8 — Why Interfaces Improve Testing
Imagine testing your service.

You don't want every service test to require:
- PostgreSQL
- A running database
- Database credentials
- Database initialization
- SQL execution

Instead, you can use a fake repository.

Fake Repository
```
package com.example.dao;

import java.sql.SQLException;
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

import com.example.model.Employee;
import com.example.repository.EmployeeRepository;

public class FakeEmployeeRepository
        implements EmployeeRepository {

    private final List<Employee> employees =
            new ArrayList<>();

    @Override
    public Optional<Employee> findById(int id) {

        return employees.stream()
                .filter(e -> e.getId() == id)
                .findFirst();
    }

    @Override
    public List<Employee> findAll() {
        return new ArrayList<>(employees);
    }

    @Override
    public int insert(Employee employee) {
        employees.add(employee);
        return 1;
    }

    @Override
    public int update(Employee employee) {

        return 0;
    }

    @Override
    public int deleteById(int id) {

        return 0;
    }
}
```

Now you can test business logic without a database.

```
FakeEmployeeRepository fakeRepository =
        new FakeEmployeeRepository();

EmployeeService service =
        new EmployeeService(fakeRepository);
```

The main benefit of abstraction is not just cleaner code.

It allows you to replace dependencies, isolate business logic, and test behavior independently.


## 7.11.9 — Connection Management Architecture
This is one of the most important design decisions in JDBC.

A common mistake is to create a new database connection inside every DAO method.

For example:
```
public Employee findById(int id) throws SQLException {

    Connection connection =
            DriverManager.getConnection(url, user, password);

    // Execute query
}
```

Problems:
- Every operation creates a new connection.
- Connection creation is expensive.
- Connection lifecycle becomes inconsistent.
- Transactions become difficult.
- Multiple DAO operations cannot easily share the same transaction.
Instead, use a central connection provider.

#### ConnectionProvider
```
package com.example.config;

import java.sql.Connection;
import java.sql.SQLException;

public interface ConnectionProvider {

    Connection getConnection() throws SQLException;
}
```

This interface defines how the application obtains a database connection.

#### DriverManagerConnectionProvider
```
package com.example.config;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class DriverManagerConnectionProvider
        implements ConnectionProvider {

    private final String url;
    private final String username;
    private final String password;

    public DriverManagerConnectionProvider(
            String url,
            String username,
            String password
    ) {

        this.url = url;
        this.username = username;
        this.password = password;
    }

    @Override
    public Connection getConnection()
            throws SQLException {

        return DriverManager.getConnection(
                url,
                username,
                password
        );
    }
}
```

Now your application can centralize connection creation.

However, remember:

DriverManager is suitable for learning and small applications. In production, use a connection pool such as HikariCP.

## 7.11.10 — HikariCP Architecture
You already studied connection pooling in Phase 7.2. Now let's understand how it fits into the architecture.

Instead of:
```
Application
   │
   ├── Create connection
   ├── Execute query
   ├── Close connection
   │
   ├── Create connection
   ├── Execute query
   └── Close connection
```

Use:
```
Application
     │
     ▼
Connection Pool
     │
 ┌───┼────┐
 ▼   ▼    ▼
 C1  C2   C3
 │   │    │
 └───┼────┘
     ▼
 PostgreSQL
```

The application requests a connection:
```
Connection connection =
        dataSource.getConnection();
```

When it closes the connection:
```
connection.close();
```

With a pooled DataSource, this generally returns the connection to the pool instead of physically destroying the database connection.

The DAO should generally not know whether the connection comes from:
- DriverManager
- HikariCP
- Another DataSource
- A testing connection provider
That abstraction is valuable.

## 7.11.11 — Who Should Manage Transactions?
This is an architectural question, not just a JDBC syntax question.

Suppose you have an operation:
Create an employee and assign that employee to a department.

This requires two SQL operations.
```
INSERT INTO employees (...);

INSERT INTO employee_departments (...);
```

Both must succeed, or both should fail.

If the transaction is managed inside each DAO method independently, you can accidentally create this situation:

```
EmployeeDAO.insert()
    └── commits employee

DepartmentDAO.assignEmployee()
    └── fails
```
Result:
Employee exists, but assignment failed.


This creates inconsistent application state.

#### Correct design
The service should coordinate the transaction when the business operation involves multiple database actions.
```
EmployeeService
       │
       ▼
 Begin Transaction
       │
       ├── EmployeeDAO.insert()
       │
       ├── DepartmentDAO.assignEmployee()
       │
       ├── Other database operation
       │
       ▼
   Commit Transaction


If something fails:
Exception
   │
   ▼
Rollback
```


## 7.11.12 — Transaction Manager Without Spring
Let's create a basic transaction utility.

#### TransactionManager.java
```
package com.example.transaction;

import java.sql.Connection;
import java.sql.SQLException;

public final class TransactionManager {

    private TransactionManager() {
    }

    public static void begin(Connection connection)
            throws SQLException {

        connection.setAutoCommit(false);
    }

    public static void commit(Connection connection)
            throws SQLException {

        connection.commit();
    }

    public static void rollback(Connection connection)
            throws SQLException {

        connection.rollback();
    }
}
```

This is a simple utility, but production transaction management requires careful handling of exceptions and connection state.

Example transaction in Service

```
public void createEmployeeWithDepartment(
        Employee employee,
        int departmentId
) throws SQLException {

    Connection connection =
            connectionProvider.getConnection();

    try {

        connection.setAutoCommit(false);

        employeeDAO.insert(employee);

        departmentDAO.assignEmployee(
                employee.getId(),
                departmentId
        );

        connection.commit();

    } catch (SQLException ex) {

        try {
            connection.rollback();
        } catch (SQLException rollbackException) {
            ex.addSuppressed(rollbackException);
        }

        throw ex;

    } finally {

        connection.close();
    }
}
```

But this example reveals a problem.

If employeeDAO and departmentDAO use different connections, the transaction won't cover both operations.

This is why transaction-aware architecture must ensure that multiple DAO operations share the same connection.

## 7.11.13 — The Shared Connection Problem
Imagine:

```
EmployeeDAO employeeDAO =
        new EmployeeDAO(connection1);

DepartmentDAO departmentDAO =
        new DepartmentDAO(connection2);
```

Then:
```
connection1.setAutoCommit(false);

employeeDAO.insert(employee);

departmentDAO.assignEmployee(
        employee.getId(),
        departmentId
);

connection1.commit();
```

This is incorrect if the department operation executes on connection2.

Your transaction controls only connection1.

The operation on connection2 may have already committed independently.

## 7.11.14 — Correct Transaction-Aware DAO Design
For a pure JDBC application, one straightforward approach is to pass the transaction's connection to the DAOs.

```
Connection connection =
        connectionProvider.getConnection();

try {

    connection.setAutoCommit(false);

    EmployeeDAO employeeDAO =
            new EmployeeDAO(connection);

    DepartmentDAO departmentDAO =
            new DepartmentDAO(connection);

    employeeDAO.insert(employee);

    departmentDAO.assignEmployee(
            employee.getId(),
            departmentId
    );

    connection.commit();

} catch (SQLException ex) {

    connection.rollback();

    throw ex;

} finally {

    connection.close();
}
```


Therefore:
```
Employee INSERT
        +
Department Assignment
        │
        ▼
    One Transaction
        │
     COMMIT


If any operation fails:
    ROLLBACK
```

This is a crucial JDBC architecture principle.

## 7.11.15 — Connection Lifecycle

A connection has a lifecycle.
Connection lifecycle

- Acquire Connection
DataSource.getConnection()

- Execute Operations
Statements, queries, updates

- Commit / Rollback
Transaction outcome

- Close Connection
Return to pool when pooled

Important rule:
```
try (Connection connection =
        dataSource.getConnection()) {

    // Database operations
}
```

Closing a pooled connection usually returns it to the pool.

Never assume:
connection.close();


means the physical connection was destroyed.

## 7.11.16 — The Generic DAO Pattern
You may notice that many DAO operations are similar.

For example:
- EmployeeDAO
- ProductDAO
- DepartmentDAO
- CustomerDAO


All might have methods such as:
- findById
- findAll
- insert
- update
- delete

This leads to the idea of a generic DAO.


#### Generic interface
```
public interface CrudRepository<T, ID> {

    T findById(ID id);

    List<T> findAll();

    int insert(T entity);

    int update(T entity);

    int deleteById(ID id);
}
```

Here:
- T - represents the entity type.
- ID - represents the ID type.

For example:
```
CrudRepository<Employee, Integer>
```

or:
```
CrudRepository<Customer, UUID>
```


#### Why this looks attractive
Instead of writing:
```
EmployeeDAO
ProductDAO
DepartmentDAO
```

You might think:
GenericDAO<Employee, Integer>
GenericDAO<Product, Long>
GenericDAO<Department, Integer>


This reduces repeated interface definitions.

However, generic DAO implementation can become problematic.

## 7.11.17 — Why Generic DAO Can Become a Bad Abstraction
Consider this:
```
public class GenericDAO<T, ID> {

    public T findById(ID id) {
        // What table?
        // What ID column?
        // What SQL?
        // What mapper?
    }
}
```

The problem is that database entities are not necessarily structurally identical.

Employee:
```
employees
id
name
email
salary
```

Product:
```
products
product_id
name
price
stock
```

Department:
```
departments
department_id
department_name
```

You now need a lot of configuration:
```
tableName
primaryKeyName
columnMapping
insertSql
updateSql
mapper
```

At some point, the generic DAO becomes more complicated than individual DAOs.
Good abstraction rule
Don't create an abstraction merely because code looks similar.

Create abstractions when they reduce complexity and make the design easier to understand, maintain, or test.


## 7.11.19 — Reusable JDBC Utilities
A good JDBC architecture reduces repeated code without hiding important behavior.

For example, this pattern appears in almost every DAO:
```
try (PreparedStatement ps =
        connection.prepareStatement(sql)) {

    ps.setInt(1, id);

    try (ResultSet rs = ps.executeQuery()) {

        if (rs.next()) {
            // Mapping
        }
    }
}
```

You can create utilities for reusable operations.

However, be careful not to create a giant utility class that hides everything.

### Utility 1 — SQL parameter binder
One simple pattern is to centralize repeated parameter binding for common types.

```
package com.example.jdbc;

import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.sql.Types;
import java.util.List;

public final class JdbcUtils {

    private JdbcUtils() {
    }

    public static void bindStrings(
            PreparedStatement ps,
            List<String> values
    ) throws SQLException {

        for (int i = 0; i < values.size(); i++) {

            String value = values.get(i);

            if (value == null) {
                ps.setNull(i + 1, Types.VARCHAR);
            } else {
                ps.setString(i + 1, value);
            }
        }
    }
}
```

Usage:
```
List<String> values =
        List.of("Rahul", "rahul@example.com");
```

For production systems, don't overgeneralize parameter binding unless it genuinely reduces complexity. JDBC types and null handling matter.

## 7.11.20 — Safe Dynamic SQL Architecture
This is a very important topic for backend developers.

Suppose you want a dynamic search:
```
SELECT *
FROM employees
WHERE department_id = ?
ORDER BY salary DESC;
```

Values can be parameterized.
```
String sql = """
        SELECT *
        FROM employees
        WHERE department_id = ?
        """;

PreparedStatement ps =
        connection.prepareStatement(sql);

ps.setInt(1, departmentId);
```

But what if you want the table name to be dynamic?
String tableName = "employees";


You cannot do this:
```
PreparedStatement ps =
        connection.prepareStatement(
                "SELECT * FROM ?"
        );

ps.setString(1, tableName);
```

SQL parameters generally represent values, not SQL identifiers.

Identifiers include:
- Table names
- Column names
- Schema names
- ORDER BY columns

Unsafe dynamic SQL
```
String sql =
        "SELECT * FROM employees ORDER BY "
        + userInput;
```

If user input is:
```
salary DESC; DROP TABLE employees;
```

You may create an SQL injection vulnerability, depending on how the SQL is executed.

Safer approach — Whitelist identifiers

```
Set<String> allowedColumns = Set.of(
        "id",
        "name",
        "email",
        "salary"
);

if (!allowedColumns.contains(sortColumn)) {
    throw new IllegalArgumentException(
            "Invalid sort column"
    );
}
```

Then:

```
String sql =
        "SELECT * FROM employees ORDER BY "
        + sortColumn;
```

This is safer because only predefined identifiers are allowed.
Security rule

PreparedStatement protects parameterized values. It does not automatically make dynamically concatenated table names, column names, or SQL fragments safe.


## 7.11.21 — The Unit of Work Pattern
You have already learned transactions.

Now let's understand the architectural concept behind them.

A Unit of Work represents a logical operation that includes multiple database actions which should be treated as one transaction.

Example:
```
Place Order
   │
   ├── Create Order
   ├── Insert Order Items
   ├── Decrease Inventory
   ├── Create Payment Record
   │
   ▼
Commit
```

If any step fails:
Rollback all database changes


In JDBC, the connection is often the transaction boundary.

```
connection.setAutoCommit(false);

try {

    orderDAO.create(order);

    orderItemDAO.insertItems(items);

    inventoryDAO.reduceStock(items);

    paymentDAO.createPayment(payment);

    connection.commit();

} catch (SQLException ex) {

    connection.rollback();

    throw ex;
}
```

The service layer is often a good place to coordinate a unit of work because it understands the complete business operation.
Unit of Work — concept
One business operation

=

Multiple SQL operations

=

One transaction boundary


## 7.11.22 — A Better JDBC Application Structure
Now let's combine the architecture.

Recommended project structure

```
src/main/java/com/example/
│
├── config/
│   ├── DatabaseConfig.java
│   └── DataSourceConfig.java
│
├── model/
│   └── Employee.java
│
├── mapper/
│   └── EmployeeMapper.java
│
├── repository/
│   └── EmployeeRepository.java
│
├── dao/
│   └── JdbcEmployeeDAO.java
│
├── service/
│   └── EmployeeService.java
│
├── transaction/
│   └── TransactionManager.java
│
├── exception/
│   ├── DatabaseException.java
│   ├── EmployeeNotFoundException.java
│   └── DuplicateEmployeeException.java
│
├── util/
│   └── JdbcUtils.java
│
└── Main.java
```

Responsibility breakdown
#### Package -	Responsibility
- config -	Database configuration and DataSource
- model -	Domain objects
- mapper -	Database result mapping
- repository -	Persistence abstraction
- dao -	JDBC implementation
- service -	Business logic
- transaction -	Transaction coordination
- exception -	Application/database exceptions
- util -	Reusable helper functionality


## 7.11.25 — JDBC Architecture Anti-Patterns
These are common mistakes that appear in real Java projects.

1. SQL inside Main
```
public static void main(String[] args) {
    // 500 lines of SQL and business logic
}
```

- Better: Main should initialize the application and coordinate startup.

2. DAO containing business rules
```
if (salary < 15000) {
    // Business rule inside DAO
}
```

- Better: Validate business rules in the Service layer.

3. Service executing SQL
String sql = "SELECT * FROM employees";


- Better: The Service calls the Repository/DAO.

4. DAO creates a new connection for every query without pooling

This creates unnecessary connection overhead and makes transaction coordination difficult.

Better: Use a DataSource and connection pool.

5. Every DAO manages its own transaction

Multiple DAOs may unintentionally use different connections and transactions.

Better: Coordinate transactions at the business-operation level.

6. Giant GenericDAO

A generic DAO may become overloaded with entity-specific SQL, mapping rules, and configuration.

Better: Keep entity-specific repositories and extract genuinely reusable functionality.

7. Catching exceptions and ignoring them
catch (SQLException ex) {
    System.out.println("Error");
}


## 7.11.27 — Questions

### Q1. What is the DAO pattern?
DAO (Data Access Object) separates database operations from the rest of the application. It encapsulates SQL and JDBC access.
### Q2. Why should SQL not be placed in the Service layer?
It mixes persistence details with business logic, making the service harder to maintain, test, and change.
### Q3. What is Dependency Injection?
Dependency Injection means providing an object with the dependencies it needs instead of allowing it to create those dependencies itself.
### Q4. What is constructor injection?
Providing dependencies through the object's constructor.
public EmployeeService(EmployeeRepository repository) {
    this.repository = repository;
}


### Q5. Why use an interface for a DAO or repository?
It separates the contract from the implementation, which can improve flexibility, testing, and dependency management.
### Q6. What is the difference between DAO and Repository?
DAO is generally persistence-oriented and database-focused. Repository is generally domain-oriented and presents a collection-like abstraction for accessing domain objects.
### Q7. What is a Unit of Work?
A Unit of Work coordinates multiple database operations that belong to one logical business operation, typically using one transaction.
### Q8. Where should transactions be managed?
At the layer that understands the complete business operation, usually the service layer or a dedicated transaction-management component.
### Q9. Why should multiple DAO operations share the same Connection in a transaction?
Because the transaction belongs to that connection. Operations executed on different connections are not automatically part of the same transaction.
### Q10. Should every DAO use a generic implementation?
No. Generic implementations are useful when they genuinely reduce duplication, but entity-specific SQL and custom operations often make specific DAOs clearer.
### Q11. Why use a connection pool?
It reduces the overhead of repeatedly creating physical database connections and manages connection reuse.
### Q12. What is the difference between closing a pooled connection and closing a physical connection?
With a pooled DataSource, close() usually returns the logical connection to the pool. The pool controls whether the underlying physical connection remains open.

7.11.28 — Practical Assignment
Build a complete Employee JDBC application using clean architecture.
Required architecture
Main
 │
 ├── DataSource
 │
 ├── EmployeeRepository
 │       └── JdbcEmployeeDAO
 │
 └── EmployeeService


Tasks
Architecture
- [ ] Create Employee model
- [ ] Create EmployeeRepository interface
- [ ] Create JdbcEmployeeDAO
- [ ] Create EmployeeMapper
- [ ] Create EmployeeService
- [ ] Create DatabaseConfig
- [ ] Create TransactionManager
- [ ] Create application Main

CRUD operations
- [ ] findById()
- [ ] findAll()
- [ ] insert()
- [ ] update()
- [ ] deleteById()

Business logic
- [ ] Validate employee name
- [ ] Validate email
- [ ] Validate salary
- [ ] Prevent invalid employee ID
- [ ] Handle employee not found
- [ ] Handle duplicate email

Advanced features
- [ ] Connection pooling
- [ ] Transaction handling
- [ ] Exception translation
- [ ] Logging
- [ ] Unit testing with fake repository
- [ ] Integration testing against PostgreSQL


Phase 7.11 — Final Revision
What you should now understand
- [ ] Layered architecture
- [ ] Model layer
- [ ] Mapper layer
- [ ] DAO layer
- [ ] Repository pattern
- [ ] Service layer
- [ ] Dependency Injection without Spring
- [ ] Constructor injection
- [ ] Interface-based design
- [ ] Connection provider
- [ ] Connection pool architecture
- [ ] Transaction boundaries
- [ ] Shared connection for transactions
- [ ] Unit of Work pattern
- [ ] Generic DAO tradeoffs
- [ ] JDBC utility design
- [ ] Safe dynamic SQL
- [ ] Unit testing
- [ ] Integration testing
- [ ] Production JDBC architecture
