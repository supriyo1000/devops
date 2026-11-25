# Phase 7.9 — JDBC Data Mapping & ResultSet to Java Objects

**Pure Java + JDBC + PostgreSQL**

In previous topics, you learned how to execute SQL queries using JDBC, handle transactions, batch operations, exceptions, and performance optimization.

Now we will learn how to convert database records into Java objects properly.

This is one of the most important concepts in building a backend because JDBC returns database data through `ResultSet`, while our application normally works with Java objects such as `Employee`, `Department`, `Order`, and `Customer`.

---

## 7.9.1 — Understanding Data Mapping

Suppose you have an `employees` table in PostgreSQL.

### PostgreSQL — employees table

| id | name | email | salary | joining_date |
|---:|---|---|---:|---|
| 101 | Rahul | rahul@example.com | 45000.50 | 2024-01-15 |
| 102 | Amit | amit@example.com | 55000.00 | 2023-06-10 |
| 103 | Priya | priya@example.com | 65000.75 | 2022-09-20 |

When you execute:

```sql
SELECT * FROM employees;
```

JDBC doesn't automatically create three `Employee` objects.

Instead, JDBC returns a `ResultSet`.

### Data Mapping Workflow

```text
PostgreSQL Result Rows
          |
          ▼
     JDBC ResultSet
 getInt(), getString(), getBigDecimal()
          |
          ▼
      Mapping Logic
 Read columns and assign values to Java fields
          |
          ▼
    Employee Java Objects
```

This process is called **Data Mapping**.

---

## 7.9.2 — What Is a ResultSet?

A `ResultSet` is an object that represents the result of a database query.

```java
ResultSet rs = preparedStatement.executeQuery();
```

Imagine the database returns:

| id | name | salary |
|---:|---|---:|
| 101 | Rahul | 45000 |
| 102 | Amit | 55000 |
| 103 | Priya | 65000 |

The ResultSet cursor initially points before the first row.

### ResultSet Cursor

```text
Initial
Before row 1

rs.next()
Row 1

rs.next()
Row 2

rs.next()
Row 3
```

Calling `next()` moves the cursor forward and returns `true` if another row exists.

```java
while (rs.next()) {
    // Access current row
}
```

### ResultSet Cursor Behavior

| Call | Result | Cursor |
|---|---|---|
| First call | true | Row 1 |
| Second call | true | Row 2 |
| Third call | true | Row 3 |
| Fourth call | false | No more rows |

**Important:** You cannot safely read a row before calling `rs.next()`.

---

## 7.9.3 — Reading Values from ResultSet

JDBC provides several getter methods.

| SQL Type | Java Getter |
|---|---|
| INTEGER | getInt() |
| BIGINT | getLong() |
| VARCHAR / TEXT | getString() |
| BOOLEAN | getBoolean() |
| NUMERIC / DECIMAL | getBigDecimal() |
| DATE | getDate() |
| TIME | getTime() |
| TIMESTAMP | getTimestamp() |
| TIMESTAMPTZ | getObject() / appropriate Java time mapping |
| UUID | getObject() |
| JSON / JSONB | getString() or driver-specific handling |

### Example

```java
int id = rs.getInt("id");

String name = rs.getString("name");

double salary = rs.getDouble("salary");
```

However, in production applications, choosing the correct Java type is important.

For example:

- salary → BigDecimal
- UUID → UUID
- DATE → LocalDate
- TIMESTAMP → LocalDateTime
- TIMESTAMPTZ → OffsetDateTime or Instant with appropriate semantics

---

## 7.9.4 — Create the Employee Model

Let's create a Java object that represents an employee row.

### Employee.java

```java
package com.example.model;

import java.math.BigDecimal;
import java.time.LocalDate;

public class Employee {

    private int id;
    private String name;
    private String email;
    private BigDecimal salary;
    private LocalDate joiningDate;

    public Employee() {
    }

    public Employee(
            int id,
            String name,
            String email,
            BigDecimal salary,
            LocalDate joiningDate
    ) {
        this.id = id;
        this.name = name;
        this.email = email;
        this.salary = salary;
        this.joiningDate = joiningDate;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public BigDecimal getSalary() {
        return salary;
    }

    public void setSalary(BigDecimal salary) {
        this.salary = salary;
    }

    public LocalDate getJoiningDate() {
        return joiningDate;
    }

    public void setJoiningDate(LocalDate joiningDate) {
        this.joiningDate = joiningDate;
    }

    @Override
    public String toString() {
        return "Employee{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", email='" + email + '\'' +
                ", salary=" + salary +
                ", joiningDate=" + joiningDate +
                '}';
    }
}
```

### Why Use BigDecimal for Salary?

Avoid this:

```java
double salary = 45000.50;
```

Floating-point values can produce precision problems when used for financial values.

For example:

```java
System.out.println(0.1 + 0.2);
```

Output:

```text
0.30000000000000004
```

Instead, use:

```java
BigDecimal salary = new BigDecimal("45000.50");
```

For money, use `BigDecimal` rather than `float` or `double`.

---

## 7.9.5 — Mapping ResultSet to an Employee

Now let's manually map one row.

### SQL

```sql
SELECT id, name, email, salary, joining_date
FROM employees;
```

### JDBC Code

```java
while (rs.next()) {

    int id = rs.getInt("id");
    String name = rs.getString("name");
    String email = rs.getString("email");
    BigDecimal salary = rs.getBigDecimal("salary");

    LocalDate joiningDate =
            rs.getDate("joining_date").toLocalDate();

    Employee employee = new Employee(
            id,
            name,
            email,
            salary,
            joiningDate
    );

    System.out.println(employee);
}
```

Here, every database row is converted into one Java `Employee` object.

### Row-to-Object Conversion

Database row:

```text
101 | Rahul | rahul@example.com | 45000.50 | 2024-01-15
```

Java Employee object:

```java
new Employee(
    101,
    "Rahul",
    "rahul@example.com",
    new BigDecimal("45000.50"),
    LocalDate.of(2024, 1, 15)
);
```

---

## 7.9.6 — Why We Need a Separate Mapper Method

Suppose you have 10 different DAO methods:

- findAll()
- findById()
- findByEmail()
- findByDepartment()
- findActiveEmployees()
- findEmployeesBySalary()

If every method contains the same mapping code, you create duplication.

Instead of repeating:

```java
int id = rs.getInt("id");
String name = rs.getString("name");
String email = rs.getString("email");
BigDecimal salary = rs.getBigDecimal("salary");
```

Create one reusable method.

### EmployeeMapper.java

```java
package com.example.mapper;

import java.sql.ResultSet;
import java.sql.SQLException;
import java.time.LocalDate;

import com.example.model.Employee;

public class EmployeeMapper {

    public static Employee map(ResultSet rs) throws SQLException {

        Employee employee = new Employee();

        employee.setId(rs.getInt("id"));
        employee.setName(rs.getString("name"));
        employee.setEmail(rs.getString("email"));
        employee.setSalary(rs.getBigDecimal("salary"));

        java.sql.Date joiningDate = rs.getDate("joining_date");

        if (joiningDate != null) {
            employee.setJoiningDate(joiningDate.toLocalDate());
        }

        return employee;
    }
}
```

Now every DAO can reuse this method.

```java
Employee employee = EmployeeMapper.map(rs);
```

### Key Concept

Mapper classes are responsible for converting data from one representation into another.

Here, we convert JDBC `ResultSet` data into Java domain objects.

---

## 7.9.7 — Implement findById() Using the Mapper

Let's build a proper DAO.

### EmployeeDAO.java

```java
package com.example.dao;

import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

import com.example.mapper.EmployeeMapper;
import com.example.model.Employee;

public class EmployeeDAO {

    private final Connection connection;

    public EmployeeDAO(Connection connection) {
        this.connection = connection;
    }

    public Employee findById(int id) throws SQLException {

        String sql = """
                SELECT id, name, email, salary, joining_date
                FROM employees
                WHERE id = ?
                """;

        try (PreparedStatement ps = connection.prepareStatement(sql)) {

            ps.setInt(1, id);

            try (ResultSet rs = ps.executeQuery()) {

                if (rs.next()) {
                    return EmployeeMapper.map(rs);
                }

                return null;
            }
        }
    }
}
```

### Execution Flow

```text
findById(101)
      |
      ▼
PreparedStatement
      |
      ▼
executeQuery()
      |
      ▼
ResultSet
      |
      ▼
EmployeeMapper.map(rs)
      |
      ▼
Returns Employee object
```

Notice that the mapper does not execute SQL.

- The DAO is responsible for SQL and JDBC execution.
- The Mapper is responsible for conversion.

---

## 7.9.8 — Mapping Multiple Rows into a List

When retrieving multiple employees, you normally need:

```java
List<Employee>
```

### DAO Method: findAll()

```java
public List<Employee> findAll() throws SQLException {

    List<Employee> employees = new ArrayList<>();

    String sql = """
            SELECT id, name, email, salary, joining_date
            FROM employees
            ORDER BY id
            """;

    try (
        PreparedStatement ps = connection.prepareStatement(sql);
        ResultSet rs = ps.executeQuery()
    ) {

        while (rs.next()) {

            Employee employee = EmployeeMapper.map(rs);

            employees.add(employee);
        }
    }

    return employees;
}
```

### How the Loop Works

```java
while (rs.next()) {
    employees.add(EmployeeMapper.map(rs));
}
```

Each iteration:

1. Moves the cursor to the next row.
2. Reads that row.
3. Creates an Employee.
4. Adds it to the list.
5. Moves to the next row.

### Result

```text
List
[
    Employee(101, "Rahul", ...),
    Employee(102, "Amit", ...),
    Employee(103, "Priya", ...)
]
```

---

## 7.9.9 — Handling SQL NULL Correctly

This is a very important JDBC topic.

Suppose the database contains:

| id | name | salary | joining_date |
|---:|---|---:|---|
| 101 | Rahul | 45000 | 2024-01-15 |
| 102 | Amit | NULL | NULL |

What happens if we do this?

```java
int departmentId = rs.getInt("department_id");
```

If the database value is `NULL`, `getInt()` returns:

```text
0
```

But 0 and NULL are not the same.

This is a problem.

### Solution 1 — wasNull()

```java
int departmentId = rs.getInt("department_id");

if (rs.wasNull()) {
    System.out.println("Department ID is NULL");
}
```

`wasNull()` checks whether the last column read using a getter contained SQL NULL.

**Important:** Call `wasNull()` immediately after the getter you want to check.

Correct:

```java
int departmentId = rs.getInt("department_id");
boolean departmentIsNull = rs.wasNull();
```

Incorrect:

```java
int departmentId = rs.getInt("department_id");

String name = rs.getString("name");

boolean departmentIsNull = rs.wasNull();
```

Here, `wasNull()` refers to the most recently read column, which is `name`, not `department_id`.

### Solution 2 — Use Wrapper Types

For nullable database columns, Java wrapper types are often better.

| Primitive | Wrapper |
|---|---|
| int | Integer |
| long | Long |
| double | Double |
| boolean | Boolean |

For example:

```java
Integer departmentId;
```

Now it can represent:

- 10
- 20
- null

A primitive `int` cannot represent null.

### Solution 3 — Generic Helper for Nullable Integers

```java
public static Integer getNullableInt(
        ResultSet rs,
        String column
) throws SQLException {

    int value = rs.getInt(column);

    if (rs.wasNull()) {
        return null;
    }

    return value;
}
```

Usage:

```java
Integer departmentId =
        getNullableInt(rs, "department_id");
```

This is useful when working with nullable numeric database fields.

---

## 7.9.10 — Mapping SQL Types to Java Types

This is extremely important for backend development.

### Recommended Mapping for PostgreSQL and Java 17

| PostgreSQL | Java |
|---|---|
| SMALLINT | Short / short |
| INTEGER | Integer / int |
| BIGINT | Long / long |
| REAL | Float |
| DOUBLE PRECISION | Double |
| NUMERIC | BigDecimal |
| VARCHAR | String |
| TEXT | String |
| BOOLEAN | Boolean |
| DATE | LocalDate |
| TIME | LocalTime |
| TIMESTAMP | LocalDateTime |
| TIMESTAMP WITH TIME ZONE | OffsetDateTime / Instant |
| UUID | UUID |
| JSON | String / JSON tree / custom type |
| JSONB | String / JSON tree / custom type |
| BYTEA | byte[] |

---

## 7.9.11 — Mapping DATE, TIME and TIMESTAMP

Modern Java applications should generally prefer `java.time` classes over legacy date/time classes.

### PostgreSQL DATE → LocalDate

```java
java.sql.Date sqlDate = rs.getDate("joining_date");

LocalDate joiningDate = null;

if (sqlDate != null) {
    joiningDate = sqlDate.toLocalDate();
}
```

### PostgreSQL TIME → LocalTime

```java
java.sql.Time sqlTime = rs.getTime("start_time");

LocalTime startTime = null;

if (sqlTime != null) {
    startTime = sqlTime.toLocalTime();
}
```

### PostgreSQL TIMESTAMP → LocalDateTime

```java
java.sql.Timestamp timestamp =
        rs.getTimestamp("created_at");

LocalDateTime createdAt = null;

if (timestamp != null) {
    createdAt = timestamp.toLocalDateTime();
}
```

### PostgreSQL TIMESTAMPTZ → Instant

For values representing an actual moment on the timeline, `Instant` is often a good choice.

```java
OffsetDateTime createdAt =
        rs.getObject("created_at", OffsetDateTime.class);

Instant instant = createdAt != null
        ? createdAt.toInstant()
        : null;
```

The exact mapping should be chosen based on whether your application needs to preserve the time zone/offset representation or simply represent the absolute moment in time.

---

## 7.9.12 — Mapping UUID

PostgreSQL:

```sql
CREATE TABLE employees (
    id UUID PRIMARY KEY,
    name VARCHAR(100)
);
```

Java:

```java
import java.util.UUID;
```

### Reading

```java
UUID id = rs.getObject("id", UUID.class);
```

### Writing

```java
ps.setObject(1, employee.getId());
```

UUIDs should generally map directly to Java's `UUID` class instead of converting them to strings unnecessarily.

---

## 7.9.13 — Mapping PostgreSQL JSON and JSONB

You will encounter this frequently when working with APIs, event data, CDC pipelines, and your Helyx data-processing projects.

Suppose PostgreSQL has:

```sql
CREATE TABLE employee_metadata (
    employee_id INTEGER,
    metadata JSONB
);
```

Example data:

```json
{
  "skills": ["Java", "Spring Boot", "PostgreSQL"],
  "experience": 5,
  "active": true
}
```

### Option 1 — Read JSON as String

```java
String metadata = rs.getString("metadata");
```

This is the simplest option.

You can pass it to another service, serialize it, or parse it using Jackson later.

### Option 2 — Convert JSON to a Jackson JsonNode

If you already use Jackson:

```java
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.ObjectMapper;

ObjectMapper objectMapper = new ObjectMapper();

String json = rs.getString("metadata");

JsonNode metadata = null;

if (json != null) {
    metadata = objectMapper.readTree(json);
}
```

Now you can access JSON values:

```java
String skill = metadata
        .get("skills")
        .get(0)
        .asText();

int experience = metadata
        .get("experience")
        .asInt();

boolean active = metadata
        .get("active")
        .asBoolean();
```

Result:

```text
skill      = Java
experience = 5
active     = true
```

For dynamic JSON data, `JsonNode` is useful because you don't need to create a Java class for every JSON structure.

---

## 7.9.14 — Mapping ENUM Values

Suppose PostgreSQL contains:

```sql
CREATE TYPE employee_status AS ENUM (
    'ACTIVE',
    'INACTIVE',
    'TERMINATED'
);
```

Employee table:

```sql
CREATE TABLE employees (
    id INTEGER PRIMARY KEY,
    name VARCHAR(100),
    status employee_status
);
```

### Java Enum

```java
public enum EmployeeStatus {
    ACTIVE,
    INACTIVE,
    TERMINATED
}
```

### Mapping from ResultSet

```java
String statusValue = rs.getString("status");

EmployeeStatus status = null;

if (statusValue != null) {
    status = EmployeeStatus.valueOf(statusValue);
}
```

For production code, remember that `Enum.valueOf()` throws an `IllegalArgumentException` if the database value doesn't match an enum constant.

If database values may change independently of your Java application, consider a safer mapping strategy.

---

## 7.9.15 — Column Names versus Column Indexes

JDBC allows both.

### Using Column Name

```java
String name = rs.getString("name");
```

### Using Column Index

```java
String name = rs.getString(2);
```

### ResultSet Column Indexes

```text
Column 1 → id

Column 2 → name

Column 3 → email

Column 4 → salary
```

JDBC column indexes start from 1, not 0.

### Which Should You Use?

Prefer column names for most application code:

```java
rs.getString("email");
```

Advantages:

- More readable.
- Less fragile when SELECT column order changes.
- Easier to maintain.
- Easier to review.

Column indexes are useful when performance-sensitive mapping has been measured and column order is tightly controlled, but readability should generally come first.

---

## 7.9.16 — Create a Reusable Mapper Architecture

Now let's organize the Employee JDBC project properly.

### Project Structure

```text
src/
└── main/
    └── java/
        └── com/example/
            ├── model/
            │   └── Employee.java
            ├── mapper/
            │   └── EmployeeMapper.java
            ├── dao/
            │   └── EmployeeDAO.java
            ├── service/
            │   └── EmployeeService.java
            ├── config/
            │   └── DatabaseConnection.java
            └── Main.java
```

### Responsibilities

| Component | Responsibility |
|---|---|
| Employee | Represents employee data |
| EmployeeMapper | Converts ResultSet → Employee |
| EmployeeDAO | Executes SQL and JDBC operations |
| EmployeeService | Business logic |
| DatabaseConnection | Manages database connection creation |
| Main | Application entry point |

This separation prevents your DAO from becoming a huge class containing SQL, mapping, business logic, and exception handling together.

---

## 7.9.17 — Complete EmployeeMapper

Let's now create a mapper you can actually reuse.

### EmployeeMapper.java

```java
package com.example.mapper;

import java.sql.ResultSet;
import java.sql.SQLException;

import com.example.model.Employee;

public final class EmployeeMapper {

    private EmployeeMapper() {
    }

    public static Employee map(ResultSet rs) throws SQLException {

        Employee employee = new Employee();

        employee.setId(rs.getInt("id"));
        employee.setName(rs.getString("name"));
        employee.setEmail(rs.getString("email"));
        employee.setSalary(rs.getBigDecimal("salary"));

        java.sql.Date joiningDate = rs.getDate("joining_date");

        employee.setJoiningDate(
                joiningDate != null
                        ? joiningDate.toLocalDate()
                        : null
        );

        return employee;
    }
}
```

### Why Did We Make This Class final?

```java
public final class EmployeeMapper
```

Because it is a utility mapper and we don't intend other classes to inherit from it.

### Why Is the Constructor Private?

```java
private EmployeeMapper() {
}
```

Because the class only provides a static mapping method and does not need to be instantiated.

Usage:

```java
Employee employee = EmployeeMapper.map(rs);
```

---

## 7.9.18 — Important Concept: DTO Mapping vs Entity Mapping

These terms are often used in backend development.

### Entity

An object representing a database-related entity, often associated with persistence and identity.

```java
Employee employee;
```

### DTO — Data Transfer Object

An object designed to transfer data between application layers or between the backend and frontend.

Example:

```java
public class EmployeeResponseDTO {

    private int id;
    private String name;
    private String email;

}
```

You might have an internal employee model:

```text
Employee
```

But the frontend only needs:

```text
EmployeeResponseDTO
```

### Why?

Because your internal model may contain:

- Salary
- Password hash
- Internal notes
- Manager details
- Audit information

But your frontend may only need:

- ID
- Name
- Email

### Typical Data Flow

```text
PostgreSQL
     |
     ▼
ResultSet
     |
     ▼
EmployeeMapper
     |
     ▼
Employee
     |
     ▼
EmployeeResponseDTO
     |
     ▼
API Response → Frontend
```

In pure JDBC, you manually implement this mapping.

Later, in Spring Boot, you will see DTO mapping patterns and libraries such as MapStruct, but the fundamental concept remains the same.

---

## 7.9.19 — Performance Considerations While Mapping

Mapping thousands of rows into Java objects is generally reasonable.

But mapping millions of rows into memory can create problems.

For example:

```java
List<Employee> employees = dao.findAll();
```

If `findAll()` retrieves 10 million rows, you may encounter:

- High memory usage
- Long garbage collection pauses
- Out-of-memory errors
- Slow response times

For large datasets, consider:

### 1. Pagination

```sql
SELECT *
FROM employees
ORDER BY id
LIMIT 1000 OFFSET 0;
```

Then fetch the next page.

### 2. Fetch Size

```java
ps.setFetchSize(500);
```

This can influence how the JDBC driver fetches rows from the database. Actual behavior depends on the driver and query configuration.

### 3. Streaming or Incremental Processing

Process one row at a time instead of collecting everything into a list.

```java
while (rs.next()) {

    Employee employee = EmployeeMapper.map(rs);

    processEmployee(employee);
}
```

This can significantly reduce memory consumption.

Streaming alone doesn't guarantee low memory usage. JDBC driver behavior, cursor configuration, auto-commit, and the way you process the results matter.

---

## 7.9.20 — Common Mistakes

### Mistake 1 — Forgetting rs.next()

Incorrect:

```java
Employee employee = EmployeeMapper.map(rs);
```

Correct:

```java
if (rs.next()) {
    Employee employee = EmployeeMapper.map(rs);
}
```

### Mistake 2 — Mapping Everything as String

```java
String salary = rs.getString("salary");
```

This can be appropriate for some use cases, but usually it is better to preserve the correct data type:

```java
BigDecimal salary = rs.getBigDecimal("salary");
```

### Mistake 3 — Ignoring SQL NULL

```java
int departmentId = rs.getInt("department_id");
```

A nullable integer can become 0 without checking `wasNull()`.

Prefer wrapper types or explicit null handling.

### Mistake 4 — Duplicating Mapping Code

Mapping logic should not be duplicated in every DAO method.

Extract reusable mapping logic into `EmployeeMapper`.

### Mistake 5 — Using double for Money

```java
double salary;
```

Prefer:

```java
BigDecimal salary;
```

### Mistake 6 — Returning Database Objects Directly to the Frontend

Don't expose internal fields automatically.

Use DTOs when you need to control what your API returns.

---

## 7.9.21 — Interview Questions

### Q1. What is a ResultSet?

A ResultSet is a JDBC object representing the rows returned by a database query. It allows you to iterate over the rows and retrieve column values.

### Q2. Why does rs.next() need to be called?

Because the ResultSet cursor initially points before the first row. `next()` moves it to the next row and returns true if a row exists.

### Q3. What is Data Mapping?

Data mapping is the process of converting data from one representation into another.

In JDBC:

\[
\text{ResultSet Row} \rightarrow \text{Java Object}
\]

### Q4. Why use a Mapper class?

To centralize mapping logic, avoid duplication, improve readability, and make DAO methods easier to maintain.

### Q5. What is the difference between getInt() and getObject()?

`getInt()` converts the value into a primitive `int`. If the database value is SQL NULL, it returns 0 and you must check `wasNull()`.

`getObject()` can return an object representation and can preserve null.

### Q6. How do you handle SQL NULL for integer columns?

Use `wasNull()` after `getInt()`, or use an appropriate nullable wrapper type such as `Integer` with explicit null handling.

### Q7. Why is BigDecimal preferred over double for financial values?

Because BigDecimal supports decimal precision suitable for monetary calculations, while floating-point types can produce precision errors.

### Q8. What is the difference between an Entity and a DTO?

An Entity represents application/database domain data, while a DTO is designed to transfer selected data between layers or systems.

### Q9. Should mapping logic be inside DAO?

Small projects may place it directly in DAO methods, but extracting a mapper is generally cleaner when mapping logic is reused or becomes complex.

### Q10. How do you map a PostgreSQL UUID into Java?

```java
UUID id = rs.getObject("id", UUID.class);
```

---

## 7.9.22 — Practice Tasks

### Employee JDBC Assignment

Complete these without using ORM.

- [ ] Create an Employee class.
- [ ] Create an EmployeeMapper class.
- [ ] Implement map(ResultSet rs).
- [ ] Implement findById(int id).
- [ ] Implement findAll().
- [ ] Map SQL NULL correctly.
- [ ] Map NUMERIC to BigDecimal.
- [ ] Map DATE to LocalDate.
- [ ] Map TIMESTAMP to LocalDateTime.
- [ ] Map PostgreSQL UUID to Java UUID.
- [ ] Read a JSONB column and parse it using Jackson.
- [ ] Create an EmployeeResponseDTO.
- [ ] Convert Employee to EmployeeResponseDTO.

---

# Phase 7.9 — Revision Checklist

### What You Should Now Understand

- [ ] What ResultSet is
- [ ] How ResultSet cursor works
- [ ] Why rs.next() is required
- [ ] How to read column values
- [ ] ResultSet → Java Object conversion
- [ ] Employee Mapper pattern
- [ ] Mapping one row
- [ ] Mapping multiple rows
- [ ] Mapping nullable columns
- [ ] wasNull()
- [ ] SQL NULL vs Java null
- [ ] BigDecimal for financial data
- [ ] PostgreSQL DATE/TIME/TIMESTAMP mapping
- [ ] UUID mapping
- [ ] JSON/JSONB mapping
- [ ] Enum mapping
- [ ] DTO vs Entity
- [ ] Mapping performance and streaming

### The Most Important Pattern From This Lesson

```java
while (rs.next()) {
    Employee employee = EmployeeMapper.map(rs);
    employees.add(employee);
}
```

This simple pattern is fundamental to manual JDBC development.

---

# Next: Phase 7.10 — JDBC Metadata & Database Introspection

We'll learn how Java can inspect databases dynamically without hardcoding table and column definitions.

Topics:

- DatabaseMetaData
- ResultSetMetaData
- Database schema discovery
- Table discovery
- Column discovery
- SQL type discovery
- Primary key discovery
- Foreign key discovery
- Constraints and indexes
- Dynamic schema inspection
- Building a metadata-driven database tool

You can use these concepts later in your JSON-to-DDL and data API projects, where Java needs to inspect database structures and build dynamic processing logic rather than relying on hardcoded schemas.