# B2.3 — PreparedStatement, NamedParameterJdbcTemplate & Dynamic SQL
Project: JDBC Banking System · Advanced SQL Operations

This lesson focuses on writing safe, flexible, maintainable SQL queries in Spring JDBC, using your existing banking project.

1. PreparedStatement vs NamedParameterJdbcTemplate
With regular JdbcTemplate, you use positional parameters (?).
String sql = """
    SELECT *
    FROM accounts
    WHERE account_id = ?
      AND status = ?
    """;

Account account = jdbcTemplate.queryForObject(
    sql,
    accountRowMapper,
    101L,
    "ACTIVE"
);


The problem becomes annoying when queries have many parameters.
SELECT *
FROM accounts
WHERE account_type = ?
  AND status = ?
  AND balance >= ?
  AND balance <= ?
  AND account_holder LIKE ?;


You must remember the exact parameter order.
jdbcTemplate.query(
    sql,
    accountRowMapper,
    "SAVINGS",
    "ACTIVE",
    new BigDecimal("1000"),
    new BigDecimal("50000"),
    "%Rahul%"
);


A parameter order mistake can introduce bugs.

2. NamedParameterJdbcTemplate
Instead of ?, we use named parameters.
WHERE account_id = :accountId


Positional parameters
WHERE account_id = ?
AND status = ?


Parameter binding depends on position.

Named parameters
WHERE account_id = :accountId
AND status = :status


Parameter binding depends on the name.

Create NamedParameterJdbcTemplate
package com.bank.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
import org.springframework.jdbc.core.JdbcTemplate;

@Configuration
public class JdbcConfig {

    @Bean
    public NamedParameterJdbcTemplate namedParameterJdbcTemplate(
            JdbcTemplate jdbcTemplate) {

        return new NamedParameterJdbcTemplate(jdbcTemplate);
    }
}


Spring Boot can also auto-configure a NamedParameterJdbcTemplate when JDBC infrastructure is available, so in many applications you can simply inject it without defining this bean manually.

3. Example — Find account using named parameters
Raw SQL
SELECT *
FROM accounts
WHERE account_id = :accountId;


Repository code
@Repository
public class AccountRepository {

    private final NamedParameterJdbcTemplate namedJdbc;

    public AccountRepository(NamedParameterJdbcTemplate namedJdbc) {
        this.namedJdbc = namedJdbc;
    }

    public Account findById(Long accountId) {

        String sql = """
                SELECT account_id, account_holder, account_type,
                       balance, status, created_at
                FROM accounts
                WHERE account_id = :accountId
                """;

        MapSqlParameterSource params = new MapSqlParameterSource()
                .addValue("accountId", accountId);

        return namedJdbc.queryForObject(
                sql,
                params,
                new AccountRowMapper()
        );
    }
}


What is MapSqlParameterSource?
It stores SQL parameter names and values.
MapSqlParameterSource params = new MapSqlParameterSource()
        .addValue("accountId", 101L)
        .addValue("status", "ACTIVE")
        .addValue("minBalance", new BigDecimal("1000"));


Think of it as:
Parameter name  →  Value

accountId      →  101
status         →  ACTIVE
minBalance     →  1000


This is particularly useful when your query has many parameters.

4. Example — Search accounts using multiple conditions
Suppose your banking application needs to search accounts using:
- Account holder name
- Account type
- Account status
- Minimum balance
Not all filters are mandatory.

For example:
Name: Rahul
Type: SAVINGS
Status: ACTIVE
Minimum balance: 10000


Or the user might search only:
Status: ACTIVE


This is where dynamic SQL becomes useful.

AccountSearchRequest.java
package com.bank.model;

import java.math.BigDecimal;

public class AccountSearchRequest {

    private String accountHolder;
    private String accountType;
    private String status;
    private BigDecimal minimumBalance;

    public AccountSearchRequest(
            String accountHolder,
            String accountType,
            String status,
            BigDecimal minimumBalance) {

        this.accountHolder = accountHolder;
        this.accountType = accountType;
        this.status = status;
        this.minimumBalance = minimumBalance;
    }

    public String getAccountHolder() {
        return accountHolder;
    }

    public String getAccountType() {
        return accountType;
    }

    public String getStatus() {
        return status;
    }

    public BigDecimal getMinimumBalance() {
        return minimumBalance;
    }
}



5. Dynamic SQL — build conditions based on input
We should not write separate queries for every combination of filters.

Instead, build the SQL dynamically based on which values are supplied.
AccountRepository — dynamic search
package com.bank.repository;

import com.bank.mapper.AccountRowMapper;
import com.bank.model.Account;
import com.bank.model.AccountSearchRequest;

import org.springframework.jdbc.core.namedparam.MapSqlParameterSource;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
import org.springframework.stereotype.Repository;

import java.util.ArrayList;
import java.util.List;

@Repository
public class AccountRepository {

    private final NamedParameterJdbcTemplate namedJdbc;

    public AccountRepository(NamedParameterJdbcTemplate namedJdbc) {
        this.namedJdbc = namedJdbc;
    }

    public List<Account> search(AccountSearchRequest request) {

        StringBuilder sql = new StringBuilder("""
                SELECT account_id, account_holder, account_type,
                       balance, status, created_at
                FROM accounts
                WHERE 1 = 1
                """);

        MapSqlParameterSource params = new MapSqlParameterSource();

        if (request.getAccountHolder() != null
                && !request.getAccountHolder().isBlank()) {

            sql.append(" AND account_holder ILIKE :accountHolder");
            params.addValue(
                    "accountHolder",
                    "%" + request.getAccountHolder().trim() + "%"
            );
        }

        if (request.getAccountType() != null
                && !request.getAccountType().isBlank()) {

            sql.append(" AND account_type = :accountType");
            params.addValue(
                    "accountType",
                    request.getAccountType()
            );
        }

        if (request.getStatus() != null
                && !request.getStatus().isBlank()) {

            sql.append(" AND status = :status");
            params.addValue(
                    "status",
                    request.getStatus()
            );
        }

        if (request.getMinimumBalance() != null) {

            sql.append(" AND balance >= :minimumBalance");
            params.addValue(
                    "minimumBalance",
                    request.getMinimumBalance()
            );
        }

        sql.append(" ORDER BY account_id");

        return namedJdbc.query(
                sql.toString(),
                params,
                new AccountRowMapper()
        );
    }
}


Why WHERE 1 = 1?
It is a convenient technique for building optional conditions.
WHERE 1 = 1
AND status = :status
AND balance >= :minimumBalance


Because 1 = 1 is always true, you can append conditions without checking whether the first condition has already been added.

It does not change the query result.

6. Dynamic search examples
The same repository method now supports different searches without writing separate SQL queries.
Example A — Search by name
AccountSearchRequest request = new AccountSearchRequest(
        "Rahul",
        null,
        null,
        null
);

List<Account> accounts = accountRepository.search(request);


Generated SQL:
SELECT account_id, account_holder, account_type,
       balance, status, created_at
FROM accounts
WHERE 1 = 1
AND account_holder ILIKE '%Rahul%'
ORDER BY account_id;


Example B — Search active savings accounts
AccountSearchRequest request = new AccountSearchRequest(
        null,
        "SAVINGS",
        "ACTIVE",
        null
);


Generated conditions:
WHERE 1 = 1
AND account_type = 'SAVINGS'
AND status = 'ACTIVE'


Example C — Find accounts with balance greater than ₹10,000
AccountSearchRequest request = new AccountSearchRequest(
        null,
        null,
        "ACTIVE",
        new BigDecimal("10000")
);


SQL conditions:
WHERE 1 = 1
AND status = 'ACTIVE'
AND balance >= 10000


This is how flexible search APIs are implemented in many real applications.

7. Advanced example — IN clause with named parameters
Suppose you want to fetch multiple accounts at once.
SELECT *
FROM accounts
WHERE account_id IN (101, 102, 103);


With regular JDBC, dynamically generating placeholders can become tedious.

With NamedParameterJdbcTemplate, you can pass a collection.
Repository method
public List<Account> findByIds(List<Long> accountIds) {

    String sql = """
            SELECT account_id, account_holder, account_type,
                   balance, status, created_at
            FROM accounts
            WHERE account_id IN (:accountIds)
            ORDER BY account_id
            """;

    MapSqlParameterSource params = new MapSqlParameterSource()
            .addValue("accountIds", accountIds);

    return namedJdbc.query(
            sql,
            params,
            new AccountRowMapper()
    );
}


Usage:
List<Long> ids = List.of(101L, 102L, 103L);

List<Account> accounts = accountRepository.findByIds(ids);


Spring expands the collection into the appropriate SQL placeholders.

Conceptually:
WHERE account_id IN (?, ?, ?)


And binds:
101
102
103


Important: Handle an empty list before executing the query. Empty IN collections can produce invalid or unexpected SQL depending on the expansion and database behavior.
if (accountIds == null || accountIds.isEmpty()) {
    return List.of();
}



8. Advanced example — Dynamic UPDATE
Suppose a bank administrator wants to update account details, but only the supplied fields should change.

Input:
Account ID: 101
New name: Rahul Das
New status: BLOCKED
Balance: Not provided


We should not accidentally overwrite the balance.
Safe dynamic update
public int updateAccount(
        Long accountId,
        String accountHolder,
        String status) {

    StringBuilder sql = new StringBuilder(
            "UPDATE accounts SET "
    );

    MapSqlParameterSource params = new MapSqlParameterSource();
    List<String> updates = new ArrayList<>();

    if (accountHolder != null && !accountHolder.isBlank()) {
        updates.add("account_holder = :accountHolder");
        params.addValue("accountHolder", accountHolder);
    }

    if (status != null && !status.isBlank()) {
        updates.add("status = :status");
        params.addValue("status", status);
    }

    if (updates.isEmpty()) {
        throw new IllegalArgumentException(
                "No update fields provided"
        );
    }

    sql.append(String.join(", ", updates));
    sql.append(" WHERE account_id = :accountId");

    params.addValue("accountId", accountId);

    return namedJdbc.update(sql.toString(), params);
}


Possible generated SQL:
UPDATE accounts
SET account_holder = :accountHolder,
    status = :status
WHERE account_id = :accountId;


Advanced consideration
Dynamic updates are useful for PATCH-like operations.

However, production code should generally use a dedicated DTO to distinguish:
- Field not supplied
- Field supplied as null
- Field supplied with a value
Those three states can have different meanings.

9. SQL Injection — why parameters matter
Unsafe SQL
String sql = "SELECT * FROM accounts WHERE account_holder = '"
        + userInput + "'";


Suppose input is:
' OR '1'='1


The resulting SQL may become:
SELECT *
FROM accounts
WHERE account_holder = '' OR '1'='1';


The condition is always true.
Safe SQL using named parameters
String sql = """
        SELECT *
        FROM accounts
        WHERE account_holder = :accountHolder
        """;

MapSqlParameterSource params = new MapSqlParameterSource()
        .addValue("accountHolder", userInput);

List<Account> accounts = namedJdbc.query(
        sql,
        params,
        new AccountRowMapper()
);


The input is treated as a value, not executable SQL.
Remember for interviews

Parameterized SQL separates SQL structure from data values. It prevents user input from being interpreted as SQL code and is a primary defense against SQL injection.


10. What parameters cannot protect
You cannot use SQL parameters for table names, column names, or keywords.

This is invalid:
SELECT * FROM :tableName


Parameters represent values, not SQL identifiers.

For example, this is unsafe:
String sql = "SELECT * FROM accounts ORDER BY " + userColumn;


If the user controls userColumn, they could inject SQL.
Correct approach: whitelist
public List<Account> findAllSorted(String sortBy) {

    Map<String, String> allowedColumns = Map.of(
            "name", "account_holder",
            "balance", "balance",
            "created", "created_at"
    );

    String column = allowedColumns.get(sortBy);

    if (column == null) {
        column = "account_id";
    }

    String sql = """
            SELECT account_id, account_holder, account_type,
                   balance, status, created_at
            FROM accounts
            ORDER BY %s
            """.formatted(column);

    return namedJdbc.query(
            sql,
            new AccountRowMapper()
    );
}


Here, the user cannot insert arbitrary SQL because only predefined columns are accepted.

Note: SQL identifiers cannot be bound with ? or :name. Use strict allowlists when dynamically selecting identifiers.

11. JdbcTemplate vs NamedParameterJdbcTemplate
Feature	JdbcTemplate	NamedParameterJdbcTemplate
Parameters	?	:parameterName
Binding	Position-based	Name-based
Readability	Good for short SQL	Better for complex SQL
Collections	Requires handling	Supports collection expansion
Dynamic queries	More difficult to manage	Usually easier
Underlying JDBC	Yes	Yes
SQL injection protection	Parameter binding	Parameter binding

JdbcTemplate
jdbcTemplate.queryForObject(
    "SELECT balance FROM accounts WHERE account_id = ?",
    BigDecimal.class,
    accountId
);


NamedParameterJdbcTemplate
namedJdbc.queryForObject(
    "SELECT balance FROM accounts WHERE account_id = :accountId",
    Map.of("accountId", accountId),
    BigDecimal.class
);


Both are valid. Choose based on query complexity and readability.

12. Interview questions
Q1. What is the difference between JdbcTemplate and NamedParameterJdbcTemplate?

JdbcTemplate uses positional placeholders (?), while NamedParameterJdbcTemplate uses named placeholders (:accountId). Named parameters improve readability and make queries easier to maintain.

Q2. Do named parameters prevent SQL injection?

Yes, when used for parameter values. SQL parameters do not protect dynamically concatenated table names, column names, or SQL keywords.

Q3. How do you build dynamic SQL in Spring JDBC?

Build the SQL structure using controlled conditions, and bind all user-provided values using MapSqlParameterSource or a parameter map.

Q4. Can I pass a List to an IN clause?

Yes, NamedParameterJdbcTemplate can expand a collection:
WHERE account_id IN (:accountIds)


Pass a collection through the parameter source.

Q5. Why use named parameters instead of positional parameters?

Named parameters are easier to read, maintain and modify, especially in SQL containing many conditions.

Q6. Can parameter binding be used for ORDER BY column names?

No. Column names are SQL identifiers, not values. Use a whitelist of allowed columns.

Revision notes
JdbcTemplate
    ? → positional parameter

NamedParameterJdbcTemplate
    :accountId → named parameter

MapSqlParameterSource
    → stores named parameters

Dynamic SQL
    → conditionally builds SQL

IN (:ids)
    → collection expansion

SQL Injection
    → bind values
    → whitelist dynamic identifiers


Practical assignment
Implement these methods in your existing banking project:
- [ ] searchAccounts() — dynamic search by name, status, account type and balance
- [ ] findByIds() — using IN (:accountIds)
- [ ] updateAccountDetails() — dynamic update of supplied fields
- [ ] findAllSorted() — safe sorting using a whitelist
Next lesson
B2.4 — JdbcTemplate CRUD, Generated Keys, Batch Processing & Performance