# 
B2.2 — RowMapper, ResultSetExtractor & Mapping Strategies
Project: JDBC Banking System · Spring JDBC

In B2.1, you learned JdbcTemplate. Now we'll focus on converting database results into Java objects efficiently.
1. Three mapping strategies
Strategy	Purpose	Best use
RowMapper<T>	Maps one row into one object	Accounts, customers, transactions
ResultSetExtractor<T>	Processes the entire ResultSet	Aggregation, custom result structures
RowCallbackHandler	Processes each row without returning a collection	Large datasets, streaming-style processing

How they differ
RowMapper
Row 1 → Account
Row 2 → Account
Row 3 → Account
          ↓
     List<Account>


ResultSetExtractor
Entire ResultSet
       ↓
Custom processing
       ↓
One final result


RowCallbackHandler
Row 1 → Process immediately
Row 2 → Process immediately
Row 3 → Process immediately




2. Banking database schema
We'll use two tables so our mapping examples have realistic relationships.
CREATE TABLE accounts (
    account_id BIGSERIAL PRIMARY KEY,
    account_holder VARCHAR(100) NOT NULL,
    account_type VARCHAR(20) NOT NULL,
    balance NUMERIC(15,2) NOT NULL DEFAULT 0,
    status VARCHAR(20) NOT NULL DEFAULT 'ACTIVE',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE transactions (
    transaction_id BIGSERIAL PRIMARY KEY,
    account_id BIGINT NOT NULL REFERENCES accounts(account_id),
    transaction_type VARCHAR(20) NOT NULL,
    amount NUMERIC(15,2) NOT NULL,
    transaction_date TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    description VARCHAR(255)
);


Insert sample data:
INSERT INTO accounts (account_holder, account_type, balance, status)
VALUES
('Rahul', 'SAVINGS', 25000.00, 'ACTIVE'),
('Amit', 'CURRENT', 80000.00, 'ACTIVE'),
('Priya', 'SAVINGS', 15000.00, 'ACTIVE'),
('Suman', 'SAVINGS', 5000.00, 'BLOCKED');

INSERT INTO transactions
(account_id, transaction_type, amount, description)
VALUES
(1, 'DEPOSIT', 5000, 'Initial deposit'),
(1, 'WITHDRAWAL', 1000, 'ATM withdrawal'),
(1, 'DEPOSIT', 2000, 'Salary deposit'),
(2, 'DEPOSIT', 10000, 'Business income'),
(2, 'WITHDRAWAL', 2500, 'Payment'),
(3, 'DEPOSIT', 3000, 'Cash deposit');



3. Project structure
banking-system/
│
├── src/main/java/com/bank/
│   ├── BankingApplication.java
│   │
│   ├── model/
│   │   ├── Account.java
│   │   ├── Transaction.java
│   │   └── AccountSummary.java
│   │
│   ├── mapper/
│   │   └── AccountRowMapper.java
│   │
│   ├── repository/
│   │   └── AccountRepository.java
│   │
│   └── service/
│       └── AccountService.java
│
└── src/main/resources/
    └── application.properties




4. Model classes
Account.java
package com.bank.model;

import java.math.BigDecimal;
import java.time.LocalDateTime;

public class Account {

    private Long accountId;
    private String accountHolder;
    private String accountType;
    private BigDecimal balance;
    private String status;
    private LocalDateTime createdAt;

    public Account(
            Long accountId,
            String accountHolder,
            String accountType,
            BigDecimal balance,
            String status,
            LocalDateTime createdAt) {

        this.accountId = accountId;
        this.accountHolder = accountHolder;
        this.accountType = accountType;
        this.balance = balance;
        this.status = status;
        this.createdAt = createdAt;
    }

    public Long getAccountId() {
        return accountId;
    }

    public String getAccountHolder() {
        return accountHolder;
    }

    public String getAccountType() {
        return accountType;
    }

    public BigDecimal getBalance() {
        return balance;
    }

    public String getStatus() {
        return status;
    }

    public LocalDateTime getCreatedAt() {
        return createdAt;
    }

    @Override
    public String toString() {
        return "Account{" +
                "accountId=" + accountId +
                ", accountHolder='" + accountHolder + '\'' +
                ", accountType='" + accountType + '\'' +
                ", balance=" + balance +
                ", status='" + status + '\'' +
                ", createdAt=" + createdAt +
                '}';
    }
}


Transaction.java
package com.bank.model;

import java.math.BigDecimal;
import java.time.LocalDateTime;

public class Transaction {

    private Long transactionId;
    private Long accountId;
    private String transactionType;
    private BigDecimal amount;
    private LocalDateTime transactionDate;
    private String description;

    public Transaction(
            Long transactionId,
            Long accountId,
            String transactionType,
            BigDecimal amount,
            LocalDateTime transactionDate,
            String description) {

        this.transactionId = transactionId;
        this.accountId = accountId;
        this.transactionType = transactionType;
        this.amount = amount;
        this.transactionDate = transactionDate;
        this.description = description;
    }

    public Long getTransactionId() {
        return transactionId;
    }

    public Long getAccountId() {
        return accountId;
    }

    public String getTransactionType() {
        return transactionType;
    }

    public BigDecimal getAmount() {
        return amount;
    }

    public LocalDateTime getTransactionDate() {
        return transactionDate;
    }

    public String getDescription() {
        return description;
    }

    @Override
    public String toString() {
        return "Transaction{" +
                "transactionId=" + transactionId +
                ", accountId=" + accountId +
                ", transactionType='" + transactionType + '\'' +
                ", amount=" + amount +
                ", transactionDate=" + transactionDate +
                ", description='" + description + '\'' +
                '}';
    }
}



5. RowMapper — mapping one row into one object
RowMapper<T> is the most commonly used mapping strategy in Spring JDBC.

Its job is simple:
Convert one database row into one Java object.

AccountRowMapper.java
package com.bank.mapper;

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
                rs.getString("account_type"),
                rs.getBigDecimal("balance"),
                rs.getString("status"),
                rs.getTimestamp("created_at").toLocalDateTime()
        );
    }
}


How it works
Suppose PostgreSQL returns:
account_id	account_holder	balance
1	Rahul	25000
2	Amit	80000
3	Priya	15000

Spring calls your mapRow() method for every row.
ResultSet
   │
   ├── Row 1 → mapRow() → Account 1
   ├── Row 2 → mapRow() → Account 2
   └── Row 3 → mapRow() → Account 3
                            │
                            ▼
                     List<Account>


You don't need to write the loop yourself.

Complete repository using RowMapper
AccountRepository.java
package com.bank.repository;

import com.bank.mapper.AccountRowMapper;
import com.bank.model.Account;
import org.springframework.dao.DataAccessException;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public class AccountRepository {

    private final JdbcTemplate jdbcTemplate;

    private final AccountRowMapper accountRowMapper =
            new AccountRowMapper();

    public AccountRepository(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    // Find one account
    public Account findById(Long accountId) {

        String sql = """
                SELECT account_id, account_holder, account_type,
                       balance, status, created_at
                FROM accounts
                WHERE account_id = ?
                """;

        return jdbcTemplate.queryForObject(
                sql,
                accountRowMapper,
                accountId
        );
    }

    // Find all accounts
    public List<Account> findAll() {

        String sql = """
                SELECT account_id, account_holder, account_type,
                       balance, status, created_at
                FROM accounts
                ORDER BY account_id
                """;

        return jdbcTemplate.query(sql, accountRowMapper);
    }

    // Find active accounts
    public List<Account> findActiveAccounts() {

        String sql = """
                SELECT account_id, account_holder, account_type,
                       balance, status, created_at
                FROM accounts
                WHERE status = ?
                ORDER BY account_id
                """;

        return jdbcTemplate.query(
                sql,
                accountRowMapper,
                "ACTIVE"
        );
    }
}


Important
The same AccountRowMapper is reused for multiple queries.

This is preferable to writing the same ResultSet conversion logic repeatedly.

6. ResultSetExtractor — processing the entire result
ResultSetExtractor<T> is different from RowMapper<T>.
RowMapper
One row → One Java object


ResultSetExtractor
Entire ResultSet → One result object


It is useful when you want:
- Aggregate results
- Custom processing
- One combined result
- Complex transformations
- Grouping rows into a custom structure

Example: Calculate total bank balance
We want to calculate:
- Total number of accounts
- Total balance
- Average balance
- Maximum balance
AccountSummary.java
package com.bank.model;

import java.math.BigDecimal;

public class AccountSummary {

    private long totalAccounts;
    private BigDecimal totalBalance;
    private BigDecimal averageBalance;
    private BigDecimal highestBalance;

    public AccountSummary(
            long totalAccounts,
            BigDecimal totalBalance,
            BigDecimal averageBalance,
            BigDecimal highestBalance) {

        this.totalAccounts = totalAccounts;
        this.totalBalance = totalBalance;
        this.averageBalance = averageBalance;
        this.highestBalance = highestBalance;
    }

    public long getTotalAccounts() {
        return totalAccounts;
    }

    public BigDecimal getTotalBalance() {
        return totalBalance;
    }

    public BigDecimal getAverageBalance() {
        return averageBalance;
    }

    public BigDecimal getHighestBalance() {
        return highestBalance;
    }

    @Override
    public String toString() {
        return "AccountSummary{" +
                "totalAccounts=" + totalAccounts +
                ", totalBalance=" + totalBalance +
                ", averageBalance=" + averageBalance +
                ", highestBalance=" + highestBalance +
                '}';
    }
}


AccountSummaryExtractor.java
package com.bank.mapper;

import com.bank.model.AccountSummary;
import org.springframework.jdbc.core.ResultSetExtractor;

import java.math.BigDecimal;
import java.sql.ResultSet;
import java.sql.SQLException;

public class AccountSummaryExtractor
        implements ResultSetExtractor<AccountSummary> {

    @Override
    public AccountSummary extractData(ResultSet rs) throws SQLException {

        if (!rs.next()) {
            return new AccountSummary(
                    0,
                    BigDecimal.ZERO,
                    BigDecimal.ZERO,
                    BigDecimal.ZERO
            );
        }

        long totalAccounts = rs.getLong("total_accounts");

        BigDecimal totalBalance =
                rs.getBigDecimal("total_balance");

        BigDecimal averageBalance =
                rs.getBigDecimal("average_balance");

        BigDecimal highestBalance =
                rs.getBigDecimal("highest_balance");

        return new AccountSummary(
                totalAccounts,
                totalBalance,
                averageBalance,
                highestBalance
        );
    }
}


Repository method
Add this method to your AccountRepository:
public AccountSummary getAccountSummary() {

    String sql = """
            SELECT
                COUNT(*) AS total_accounts,
                COALESCE(SUM(balance), 0) AS total_balance,
                COALESCE(AVG(balance), 0) AS average_balance,
                COALESCE(MAX(balance), 0) AS highest_balance
            FROM accounts
            """;

    return jdbcTemplate.query(
            sql,
            new AccountSummaryExtractor()
    );
}


Expected result
AccountSummary{
    totalAccounts=4,
    totalBalance=125000.00,
    averageBalance=31250.00,
    highestBalance=80000.00
}


Here, PostgreSQL does the aggregation, and ResultSetExtractor converts the final result into one Java object.

Advanced principle: If SQL can efficiently perform aggregation, prefer calculating it inside the database instead of loading every record and calculating in Java.

7. RowCallbackHandler — process rows without building a List
Suppose your bank has 2 million transaction records.

If you use:
List<Transaction> transactions = jdbcTemplate.query(...);


You may load a very large number of objects into memory.

If your goal is to process every row, calculate something, write to a file, or export records, RowCallbackHandler can be useful.

Unlike RowMapper, it doesn't return a collection.

TransactionRowMapper.java
package com.bank.mapper;

import com.bank.model.Transaction;
import org.springframework.jdbc.core.RowMapper;

import java.sql.ResultSet;
import java.sql.SQLException;

public class TransactionRowMapper implements RowMapper<Transaction> {

    @Override
    public Transaction mapRow(ResultSet rs, int rowNum)
            throws SQLException {

        return new Transaction(
                rs.getLong("transaction_id"),
                rs.getLong("account_id"),
                rs.getString("transaction_type"),
                rs.getBigDecimal("amount"),
                rs.getTimestamp("transaction_date").toLocalDateTime(),
                rs.getString("description")
        );
    }
}


Example: Calculate total transaction amount while processing
package com.bank.repository;

import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowCallbackHandler;
import org.springframework.stereotype.Repository;

import java.math.BigDecimal;
import java.util.concurrent.atomic.AtomicLong;

@Repository
public class TransactionReportRepository {

    private final JdbcTemplate jdbcTemplate;

    public TransactionReportRepository(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public BigDecimal calculateTotalDeposits() {

        String sql = """
                SELECT amount
                FROM transactions
                WHERE transaction_type = 'DEPOSIT'
                """;

        BigDecimal[] total = { BigDecimal.ZERO };

        RowCallbackHandler handler = rs -> {
            BigDecimal amount = rs.getBigDecimal("amount");

            total[0] = total[0].add(amount);
        };

        jdbcTemplate.query(sql, handler);

        return total[0];
    }

    public long countTransactions() {

        String sql = """
                SELECT transaction_id
                FROM transactions
                """;

        AtomicLong counter = new AtomicLong();

        jdbcTemplate.query(sql, rs -> {
            counter.incrementAndGet();
        });

        return counter.get();
    }
}


Why use RowCallbackHandler?
You process each row as it arrives through the JDBC processing loop rather than creating a List<Transaction>.

However, it does not automatically mean the entire database query is streamed with minimal memory. JDBC driver fetch behavior, fetch size, cursor settings and transaction configuration also affect memory usage. We will cover that in the performance lessons.

8. Mapping different SQL data types
This is important when working with PostgreSQL.
PostgreSQL	Java	JDBC
BIGINT	Long	getLong()
INTEGER	Integer	getInt()
VARCHAR	String	getString()
NUMERIC	BigDecimal	getBigDecimal()
BOOLEAN	Boolean	getBoolean()
DATE	LocalDate	getDate()
TIMESTAMP	LocalDateTime	getTimestamp()
TIMESTAMPTZ	OffsetDateTime / Instant	Driver-dependent mapping

Example: nullable numeric values
This code:
long id = rs.getLong("account_id");


If the database value is NULL, JDBC primitive getter methods may return 0.

To detect null:
long id = rs.getLong("account_id");

if (rs.wasNull()) {
    // Database value was NULL
}


Or use an object-returning approach when nullable data is expected:
Long id = rs.getObject("account_id", Long.class);


Interview point: JDBC primitive getters like getInt() and getLong() cannot directly represent SQL NULL as Java null. Use wasNull() or nullable object types.

9. Advanced example — map joined rows
Suppose we want account information along with its transaction information.
SELECT
    a.account_id,
    a.account_holder,
    a.balance,
    t.transaction_id,
    t.transaction_type,
    t.amount
FROM accounts a
LEFT JOIN transactions t
    ON a.account_id = t.account_id
WHERE a.account_id = ?;


This query can return:
account_id	account_holder	transaction_id	transaction_type
1	Rahul	1	DEPOSIT
1	Rahul	2	WITHDRAWAL
1	Rahul	3	DEPOSIT

Notice: the same account appears multiple times.

A simple RowMapper<Account> would create three separate Account objects.

But what if we want this structure?
Account
 ├── accountId: 1
 ├── holder: Rahul
 └── transactions
      ├── Transaction 1
      ├── Transaction 2
      └── Transaction 3


This is a good use case for ResultSetExtractor.

Instead of mapping every row independently, we process the complete ResultSet and build one parent object with its child collection.

This pattern is commonly used for:
- Parent-child data
- One-to-many relationships
- Aggregated response structures
- Custom DTOs
We'll implement complex parent-child mapping in a dedicated lesson on advanced mapping and relationships.

10. Which mapping strategy should you choose?
Requirement	Recommended
Fetch one account	queryForObject() + RowMapper
Fetch 100 accounts	query() + RowMapper
Map each row into DTO	RowMapper
Aggregate entire ResultSet	ResultSetExtractor
Build nested parent-child structure	ResultSetExtractor
Process rows without building a list	RowCallbackHandler
Count, sum, average	SQL aggregation + queryForObject()

Interview questions
1. Does RowMapper create a new object for every row?

Yes, normally you create and return a new object from each mapRow() invocation.

2. What is the difference between RowMapper and ResultSetExtractor?

RowMapper maps individual rows. ResultSetExtractor processes the entire ResultSet and returns one final result.

3. Why not use RowMapper for everything?

It's excellent for independent rows, but complex results such as grouped parent-child structures may require processing the entire ResultSet.

4. Is RowCallbackHandler more memory-efficient?

It avoids building a result collection, which can reduce application-side object memory usage. But true large-scale streaming also depends on JDBC fetch size, driver behavior, and cursor configuration.

5. Can I use lambdas instead of creating RowMapper classes?

Yes.
List<String> holders = jdbcTemplate.query(
    "SELECT account_holder FROM accounts",
    (rs, rowNum) -> rs.getString("account_holder")
);


This is useful for simple mapping. For complex mapping, a separate mapper class improves readability and reusability.

6. Should I put business logic inside RowMapper?

No. RowMapper should primarily transform database rows into Java objects. Business rules belong in the service layer.

Final revision
RowMapper<T>
    ↓
One row → One object

ResultSetExtractor<T>
    ↓
Entire ResultSet → One final result

RowCallbackHandler
    ↓
Process each row → No returned collection


Today's practical task
Implement these four methods in your existing banking repository:
- [ ] findById() using RowMapper
- [ ] findAll() using RowMapper
- [ ] getAccountSummary() using ResultSetExtractor
- [ ] calculateTotalDeposits() using RowCallbackHandler
Make sure each method works against your PostgreSQL database.

Next: B2.3 — PreparedStatement, NamedParameterJdbcTemplate & Dynamic SQL

We'll learn named parameters, handling many query parameters, dynamic filters, safe SQL construction, and more advanced banking queries.