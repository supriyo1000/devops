# B2.1 — Spring JDBC & JdbcTemplate
```
Project: JDBC Banking System
```

### 1. Why JdbcTemplate exists
In your previous JDBC code, you handled many repetitive things manually:

```
Connection connection = null;
PreparedStatement ps = null;
ResultSet rs = null;

try {
    connection = dataSource.getConnection();

    ps = connection.prepareStatement(
        "SELECT * FROM accounts WHERE account_id = ?");

    ps.setLong(1, accountId);

    rs = ps.executeQuery();

    // Mapping logic

} catch (SQLException e) {
    // Exception handling
} finally {
    // Close resources
}
```

The problem isn't JDBC itself. The problem is repetitive infrastructure code.

Spring JDBC handles much of that infrastructure for you.


|Raw JDBC|	Spring JDBC|
|--------|-------------|
|Acquire connection manually|	Managed by Spring|
|Create PreparedStatement|	Handled|
|Bind parameters|	Simplified|
|Execute SQL|	Simplified|
|Close resources|	Automatic|
|Handle SQLException|	Translated to Spring exceptions|
|Map ResultSet|	Using RowMapper|

Spring JDBC still uses JDBC underneath. It doesn't replace the JDBC driver.


### 2. How JdbcTemplate works internally
Your application code

```
jdbcTemplate.query(...);

↓

JdbcTemplate
- Gets connection from DataSource
- Creates PreparedStatement
- Binds parameters
- Executes SQL
- Maps ResultSet
- Releases resources
↓

HikariCP → PostgreSQL JDBC Driver → PostgreSQL
```

Spring manages JDBC resource handling. You still control the SQL.

### 3. Add Spring JDBC to your banking project
If you're using Maven, add the following dependency:
```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>


Your existing PostgreSQL driver remains:
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <scope>runtime</scope>
</dependency>
```

You already know how to configure the database and HikariCP, so we won't repeat that.

### 4. Create your Account model
Assuming your banking table contains these columns:
```
CREATE TABLE accounts (
    account_id BIGSERIAL PRIMARY KEY,
    account_holder VARCHAR(100) NOT NULL,
    balance NUMERIC(12,2) NOT NULL
);
```

#### Java model:

```
package com.bank.model;

import java.math.BigDecimal;

public class Account {

    private Long accountId;
    private String accountHolder;
    private BigDecimal balance;

    public Account(Long accountId, String accountHolder, BigDecimal balance) {
        this.accountId = accountId;
        this.accountHolder = accountHolder;
        this.balance = balance;
    }

    public Long getAccountId() {
        return accountId;
    }

    public String getAccountHolder() {
        return accountHolder;
    }

    public BigDecimal getBalance() {
        return balance;
    }
}
```

Important for banking: Use BigDecimal for money, never double or float.

### 5. JdbcTemplate — first query
Spring Boot automatically creates a JdbcTemplate bean when the required JDBC infrastructure is configured.

Inject it into your class:

```
@Repository
public class AccountRepository {

    private final JdbcTemplate jdbcTemplate;

    public AccountRepository(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }
}
```

This is constructor injection.

You don't manually create:
- Connection
- PreparedStatement
- ResultSet
Spring handles those internally.

#### Example 1 — Get account balance
Raw SQL:
```
SELECT balance
FROM accounts
WHERE account_id = ?;
```

#### With JdbcTemplate:
```
public BigDecimal getBalance(Long accountId) {

    return jdbcTemplate.queryForObject(
        "SELECT balance FROM accounts WHERE account_id = ?",
        BigDecimal.class,
        accountId
    );
}
```

#### Understand queryForObject()
```
queryForObject(SQL, ExpectedType, Parameters...)
```

It is used when you expect one value.

Examples:
```
// Balance

BigDecimal balance = jdbcTemplate.queryForObject(
    "SELECT balance FROM accounts WHERE account_id = ?",
    BigDecimal.class,
    101L
);

// Count accounts
Integer count = jdbcTemplate.queryForObject(
    "SELECT COUNT(*) FROM accounts",
    Integer.class
);

// Get account holder
String name = jdbcTemplate.queryForObject(
    "SELECT account_holder FROM accounts WHERE account_id = ?",
    String.class,
    101L
);
```

#### What if no account exists?
queryForObject() expects a result. If no matching row is found, Spring throws an exception, commonly:
```
EmptyResultDataAccessException
```

This is one example of Spring translating JDBC exceptions into its own exception hierarchy.

#### 6. Example 2 — Map ResultSet to Account
Previously, in raw JDBC, you wrote:

```
Account account = new Account(
    rs.getLong("account_id"),
    rs.getString("account_holder"),
    rs.getBigDecimal("balance")
);
```

With Spring JDBC, we use RowMapper.
#### Create AccountRowMapper
```
package com.bank.repository.mapper;

import com.bank.model.Account;
import org.springframework.jdbc.core.RowMapper;

import java.sql.ResultSet;
import java.sql.SQLException;

public class AccountRowMapper implements RowMapper<Account> {

    @Override
    public Account mapRow(ResultSet rs, int rowNum) throws SQLException {

        return new Account(
            rs.getLong("account_id"),
            rs.getString("account_holder"),
            rs.getBigDecimal("balance")
        );
    }
}
```

#### What is RowMapper?
RowMapper<Account>


It tells Spring:
For every database row, convert that row into an Account object.

For example, PostgreSQL returns:
```
account_id	account_holder	balance
101	Rahul	5000
102	Amit	8000
```
The mapper converts each row:
```
Database Row
      ↓
AccountRowMapper
      ↓
Account Java Object
```


#### Example 3 — Find account by ID
Inside AccountRepository:

```
public Account findById(Long accountId) {

    String sql = """
        SELECT account_id, account_holder, balance
        FROM accounts
        WHERE account_id = ?
        """;

    return jdbcTemplate.queryForObject(
        sql,
        new AccountRowMapper(),
        accountId
    );
}
```

Notice the difference:
queryForObject()


For a single expected row.
query()


For multiple rows.

#### 7. Example 4 — Fetch all accounts
Use query() when multiple rows are expected.
public List<Account> findAll() {

    String sql = """
        SELECT account_id, account_holder, balance
        FROM accounts
        ORDER BY account_id
        """;

    return jdbcTemplate.query(sql, new AccountRowMapper());
}


JdbcTemplate.query() returns a List<T>.
List<Account> accounts = accountRepository.findAll();


If no records exist, it returns an empty list rather than throwing an empty-result exception.

8. Example 5 — Create an account
In JDBC, you previously used executeUpdate().

In Spring JDBC, use:
jdbcTemplate.update()


public int createAccount(String holder, BigDecimal initialBalance) {

    String sql = """
        INSERT INTO accounts (account_holder, balance)
        VALUES (?, ?)
        """;

    return jdbcTemplate.update(sql, holder, initialBalance);
}


The return value represents the number of affected rows.
int rows = accountRepository.createAccount(
    "Supriyo",
    new BigDecimal("5000.00")
);

System.out.println(rows); // 1


update() is used for
- INSERT
- UPDATE
- DELETE
It returns the number of affected rows.

9. Example 6 — Deposit money
public int deposit(Long accountId, BigDecimal amount) {

    String sql = """
        UPDATE accounts
        SET balance = balance + ?
        WHERE account_id = ?
        """;

    return jdbcTemplate.update(sql, amount, accountId);
}


Usage:
accountRepository.deposit(
    101L,
    new BigDecimal("1000.00")
);


SQL executed:
UPDATE accounts
SET balance = balance + 1000
WHERE account_id = 101;


Why is this better than reading and updating the balance in Java?

Instead of:
Read balance → Java calculation → Update balance


We perform:
balance = balance + ?


The calculation occurs directly in the database, reducing the risk of concurrent updates overwriting each other.

However, business validations and transaction-level protection still matter.

10. Example 7 — Withdraw money
public int withdraw(Long accountId, BigDecimal amount) {

    String sql = """
        UPDATE accounts
        SET balance = balance - ?
        WHERE account_id = ?
          AND balance >= ?
        """;

    return jdbcTemplate.update(
        sql,
        amount,
        accountId,
        amount
    );
}


This is an important database-level safeguard.

If the account has insufficient funds, the update affects zero rows.
int rows = accountRepository.withdraw(
    101L,
    new BigDecimal("1000.00")
);

if (rows == 0) {
    System.out.println("Account not found or insufficient balance");
}


Advanced point: The condition balance >= ? is safer than checking the balance in Java and updating it separately because it combines the validation and update into one SQL statement.

11. Example 8 — Update account holder
public int updateAccountHolder(Long accountId, String newName) {

    String sql = """
        UPDATE accounts
        SET account_holder = ?
        WHERE account_id = ?
        """;

    return jdbcTemplate.update(sql, newName, accountId);
}



12. Example 9 — Delete an account
public int deleteAccount(Long accountId) {

    String sql = """
        DELETE FROM accounts
        WHERE account_id = ?
        """;

    return jdbcTemplate.update(sql, accountId);
}


Again, the return value tells you what happened:
int deleted = accountRepository.deleteAccount(101L);

if (deleted == 1) {
    System.out.println("Account deleted");
} else {
    System.out.println("Account not found");
}



13. Repository vs Service — banking architecture
Don't put all banking logic directly inside your repository.
Architecture
Controller
   │
   ▼
AccountService
   │  Business Logic
   ▼
AccountRepository
   │  SQL + JdbcTemplate
   ▼
PostgreSQL



AccountService
@Service
public class AccountService {

    private final AccountRepository repository;

    public AccountService(AccountRepository repository) {
        this.repository = repository;
    }

    public BigDecimal getBalance(Long accountId) {
        return repository.getBalance(accountId);
    }

    public void deposit(Long accountId, BigDecimal amount) {

        if (amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }

        int rows = repository.deposit(accountId, amount);

        if (rows == 0) {
            throw new IllegalArgumentException("Account not found");
        }
    }

    public void withdraw(Long accountId, BigDecimal amount) {

        if (amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }

        int rows = repository.withdraw(accountId, amount);

        if (rows == 0) {
            throw new IllegalArgumentException(
                "Account not found or insufficient balance"
            );
        }
    }
}


Repository → Database operations

Service → Business rules

This separation is extremely important in real Spring Boot projects and interviews.

14. JdbcTemplate methods — must remember
Method	Purpose	Example
queryForObject()	Fetch one value/object	Balance, count, single account
query()	Fetch multiple records	All accounts
update()	INSERT, UPDATE, DELETE	Deposit, withdraw
execute()	Execute SQL/DDL operations	Create table, database operations
batchUpdate()	Execute multiple operations efficiently	Bulk deposits/imports

Example:
Integer totalAccounts = jdbcTemplate.queryForObject(
    "SELECT COUNT(*) FROM accounts",
    Integer.class
);


List<Account> accounts = jdbcTemplate.query(
    "SELECT * FROM accounts",
    new AccountRowMapper()
);


int rows = jdbcTemplate.update(
    "UPDATE accounts SET balance = balance + ? WHERE account_id = ?",
    new BigDecimal("500"),
    101L
);



15. Interview preparation
Q1. What is JdbcTemplate?
JdbcTemplate is a Spring JDBC class that simplifies database operations by managing connection acquisition, statement creation, SQL execution, resource cleanup and exception translation.

Q2. Does JdbcTemplate replace JDBC?
No.

JdbcTemplate uses JDBC internally. It reduces boilerplate code while allowing us to write SQL directly.

Q3. Why use JdbcTemplate instead of raw JDBC?
- Less boilerplate
- Automatic resource cleanup
- Simplified parameter binding
- ResultSet mapping support
- Exception translation
- Integration with Spring transaction management

Q4. What is the difference between query() and queryForObject()?
- query() is generally used to retrieve multiple rows, returning a collection.
- queryForObject() is used when a single result is expected.

Q5. What is RowMapper?
RowMapper<T> converts an individual ResultSet row into a Java object.
public interface RowMapper<T> {
    T mapRow(ResultSet rs, int rowNum) throws SQLException;
}



Q6. Why does JdbcTemplate use PreparedStatement?
To support parameterized SQL, reduce SQL injection risks, and efficiently handle parameter binding.
jdbcTemplate.queryForObject(
    "SELECT balance FROM accounts WHERE account_id = ?",
    BigDecimal.class,
    accountId
);


Never construct SQL by concatenating user input.

Q7. Does JdbcTemplate manage transactions automatically?
It manages JDBC resources, but transaction boundaries are generally managed separately through Spring's transaction management, such as @Transactional.

Q8. What happens when a query returns no rows?
For queryForObject(), Spring typically throws EmptyResultDataAccessException.

For query(), an empty result generally produces an empty list.

Quick revision
JdbcTemplate
    │
    ├── queryForObject() → Single result
    ├── query()         → Multiple results
    ├── update()        → INSERT / UPDATE / DELETE
    ├── batchUpdate()   → Bulk operations
    └── execute()       → SQL execution


RowMapper
    ↓
ResultSet row
    ↓
Java Object


Repository: SQL and data access.

Service: Business logic.

JdbcTemplate: Simplifies JDBC operations.

Next lesson: B2.2 — RowMapper, ResultSetExtractor & Mapping Strategies
We'll compare RowMapper, ResultSetExtractor, and RowCallbackHandler with actual banking examples, including when each is appropriate, performance considerations, and interview questions.