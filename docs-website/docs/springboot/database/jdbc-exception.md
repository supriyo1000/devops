# B2.5 — Spring JDBC Exception Translation & Production Error Handling
In raw JDBC, database errors generally come through SQLException. In Spring JDBC, JdbcTemplate translates these errors into Spring's DataAccessException hierarchy.

This gives you cleaner, more consistent error handling without depending heavily on database-specific exceptions.
1. JDBC vs Spring JDBC exception handling
Raw JDBC
try {
    // JDBC operation
} catch (SQLException e) {
    System.out.println(e.getSQLState());
    System.out.println(e.getErrorCode());
}


Problems:
- SQL exceptions are checked exceptions.
- You need repetitive try-catch blocks.
- Error codes differ across databases.
- Database errors can leak into business logic.
Spring JDBC
try {
    jdbcTemplate.update(sql, parameters);
} catch (DataAccessException e) {
    // Spring database exception
}


Spring translates underlying SQL exceptions into meaningful runtime exceptions.
Exception translation flow
PostgreSQL / JDBC SQLException


JdbcTemplate Exception Translator


Spring DataAccessException



2. Important Spring JDBC exceptions
These are the exceptions you should recognize in production.
Exception	Typical cause
DataAccessException	Base class for Spring data-access exceptions
DuplicateKeyException	Duplicate value violates unique constraint
DataIntegrityViolationException	Constraint violation, invalid data
EmptyResultDataAccessException	Expected one row, but zero rows found
IncorrectResultSizeDataAccessException	Query returned unexpected number of rows
BadSqlGrammarException	Invalid SQL syntax or SQL statement
QueryTimeoutException	Query exceeded timeout
TransientDataAccessResourceException	Temporary database/resource problem
CannotAcquireLockException	Lock acquisition failure
DataAccessResourceFailureException	Database connection/resource failure

Most of these extend DataAccessException.

3. Exception hierarchy

DataAccessException
│
├── NonTransientDataAccessException
│   ├── DataIntegrityViolationException
│   │   └── DuplicateKeyException
│   ├── BadSqlGrammarException
│   └── InvalidDataAccessApiUsageException
│
├── TransientDataAccessException
│   ├── QueryTimeoutException
│   ├── TransientDataAccessResourceException
│   └── CannotAcquireLockException
│
└── RecoverableDataAccessException



The exact hierarchy may differ slightly between specific exceptions, but the key idea is:
- Non-transient: repeating the same operation is unlikely to succeed without changing something.
- Transient: the operation might succeed if retried later.
- Recoverable: the application may be able to recover from the failure.

4. Handling EmptyResultDataAccessException
Suppose we search for an account.
public Account findById(Long accountId) {

    String sql = """
        SELECT account_id, account_holder, account_type,
               balance, status, created_at
        FROM accounts
        WHERE account_id = ?
        """;

    return jdbcTemplate.queryForObject(
        sql,
        rowMapper,
        accountId
    );
}


If the account doesn't exist, queryForObject() throws:
EmptyResultDataAccessException


Better repository handling
public Account findById(Long accountId) {

    String sql = """
        SELECT account_id, account_holder, account_type,
               balance, status, created_at
        FROM accounts
        WHERE account_id = ?
        """;

    try {
        return jdbcTemplate.queryForObject(
            sql,
            rowMapper,
            accountId
        );

    } catch (EmptyResultDataAccessException e) {
        return null;
    }
}


However, returning null isn't always the best design because callers must remember to check for it.

A better design is to let the service handle the missing account using a domain exception.

5. Create a custom business exception
A database exception and a business exception are not the same thing.

For example:
- EmptyResultDataAccessException → database query returned no row.
- AccountNotFoundException → business operation requires an account that doesn't exist.
AccountNotFoundException.java
package com.bank.exception;

public class AccountNotFoundException extends RuntimeException {

    public AccountNotFoundException(Long accountId) {
        super("Account not found with ID: " + accountId);
    }
}


Repository
Let the database-related exception occur naturally.
public Account findById(Long accountId) {

    String sql = """
        SELECT account_id, account_holder, account_type,
               balance, status, created_at
        FROM accounts
        WHERE account_id = ?
        """;

    return jdbcTemplate.queryForObject(
        sql,
        rowMapper,
        accountId
    );
}


Service
Translate the repository failure into a business-level exception.
package com.bank.service;

import com.bank.model.Account;
import com.bank.repository.AccountRepository;
import com.bank.exception.AccountNotFoundException;

import org.springframework.dao.EmptyResultDataAccessException;
import org.springframework.stereotype.Service;

@Service
public class AccountService {

    private final AccountRepository accountRepository;

    public AccountService(AccountRepository accountRepository) {
        this.accountRepository = accountRepository;
    }

    public Account getAccount(Long accountId) {

        try {
            return accountRepository.findById(accountId);

        } catch (EmptyResultDataAccessException e) {
            throw new AccountNotFoundException(accountId);
        }
    }
}


Architecture principle:

Repository handles database operations. Service handles business interpretation of failures.

6. Handling duplicate accounts
Suppose the bank requires account holders to have unique account numbers.

Database constraint:
ALTER TABLE accounts
ADD CONSTRAINT uq_account_number
UNIQUE (account_number);


When a duplicate value is inserted, PostgreSQL returns a constraint violation. Spring translates it into a DuplicateKeyException.
Custom exception
package com.bank.exception;

public class DuplicateAccountException extends RuntimeException {

    public DuplicateAccountException(String message) {
        super(message);
    }
}


Repository
public Long createAccount(Account account) {

    String sql = """
        INSERT INTO accounts
            (account_number, account_holder, account_type, balance, status)
        VALUES (?, ?, ?, ?, ?)
        """;

    KeyHolder keyHolder = new GeneratedKeyHolder();

    try {
        jdbcTemplate.update(connection -> {

            PreparedStatement ps = connection.prepareStatement(
                sql,
                new String[]{"account_id"}
            );

            ps.setString(1, account.getAccountNumber());
            ps.setString(2, account.getAccountHolder());
            ps.setString(3, account.getAccountType());
            ps.setBigDecimal(4, account.getBalance());
            ps.setString(5, account.getStatus());

            return ps;

        }, keyHolder);

        return keyHolder.getKey().longValue();

    } catch (DuplicateKeyException e) {
        throw e;
    }
}


The repository doesn't need to catch and rethrow the exact same exception.
Better service-level translation
public Long createAccount(Account account) {

    try {
        return accountRepository.createAccount(account);

    } catch (DuplicateKeyException e) {

        throw new DuplicateAccountException(
            "Account number already exists"
        );
    }
}


Now your business layer does not need to expose database-specific concepts.

7. Don't catch and swallow exceptions
Bad practice
public void deposit(Long accountId, BigDecimal amount) {

    try {
        accountRepository.deposit(accountId, amount);

    } catch (Exception e) {
        System.out.println("Something went wrong");
    }
}


Why is this dangerous?
- The caller thinks the operation succeeded.
- Transactions may not behave as expected.
- The original cause is lost.
- Monitoring cannot properly detect the failure.
Better
public void deposit(Long accountId, BigDecimal amount) {

    int rowsUpdated = accountRepository.deposit(accountId, amount);

    if (rowsUpdated == 0) {
        throw new AccountNotFoundException(accountId);
    }
}


Let unexpected exceptions propagate or translate them while preserving the cause.

8. @Transactional and exception handling
Spring transactions behave differently depending on the exception type.

By default, Spring rolls back when a transactional method throws:
- RuntimeException
- Error
Checked exceptions do not automatically trigger rollback by default.
Example
@Transactional
public void transfer(Long fromId, Long toId, BigDecimal amount) {

    accountRepository.withdraw(fromId, amount);

    accountRepository.deposit(toId, amount);

    throw new RuntimeException("Simulated failure");
}


If this RuntimeException occurs, the transaction is rolled back, so both operations are reverted.

Important

Don't catch the exception and suppress it inside the transactional method. If you swallow the exception, Spring may not see the failure and may commit the transaction.
Checked exception rollback
@Transactional(rollbackFor = Exception.class)
public void transfer(Long fromId, Long toId, BigDecimal amount)
        throws Exception {

    accountRepository.withdraw(fromId, amount);
    accountRepository.deposit(toId, amount);

    // If a checked exception occurs, rollbackFor ensures rollback.
}


For production code, transaction boundaries should normally be at the service layer, not individual repository methods.

9. Production exception-handling architecture
Controller
HTTP request / response

Service
Business rules and exception translation

Repository
JdbcTemplate / SQL / database access

PostgreSQL


Exceptions move upward until a layer knows how to handle them.
Layer	Responsibility
Repository	Perform SQL, allow Spring data exceptions
Service	Translate into business exceptions
Controller	Handle request/response
Global Exception Handler	Convert exceptions into HTTP responses


10. Global exception handling with @RestControllerAdvice
Instead of writing exception handling inside every controller, we can create one centralized handler.
Error response DTO
ApiError.java
package com.bank.dto;

import java.time.LocalDateTime;

public class ApiError {

    private LocalDateTime timestamp;
    private int status;
    private String error;
    private String message;
    private String path;

    public ApiError(
            LocalDateTime timestamp,
            int status,
            String error,
            String message,
            String path) {

        this.timestamp = timestamp;
        this.status = status;
        this.error = error;
        this.message = message;
        this.path = path;
    }

    public LocalDateTime getTimestamp() {
        return timestamp;
    }

    public int getStatus() {
        return status;
    }

    public String getError() {
        return error;
    }

    public String getMessage() {
        return message;
    }

    public String getPath() {
        return path;
    }
}


Global exception handler
GlobalExceptionHandler.java
package com.bank.exception;

import com.bank.dto.ApiError;

import jakarta.servlet.http.HttpServletRequest;

import org.springframework.dao.DataAccessException;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

import java.time.LocalDateTime;

@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(AccountNotFoundException.class)
    public ResponseEntity<ApiError> handleAccountNotFound(
            AccountNotFoundException ex,
            HttpServletRequest request) {

        ApiError error = new ApiError(
            LocalDateTime.now(),
            404,
            "NOT_FOUND",
            ex.getMessage(),
            request.getRequestURI()
        );

        return ResponseEntity
                .status(HttpStatus.NOT_FOUND)
                .body(error);
    }

    @ExceptionHandler(DuplicateAccountException.class)
    public ResponseEntity<ApiError> handleDuplicateAccount(
            DuplicateAccountException ex,
            HttpServletRequest request) {

        ApiError error = new ApiError(
            LocalDateTime.now(),
            409,
            "CONFLICT",
            ex.getMessage(),
            request.getRequestURI()
        );

        return ResponseEntity
                .status(HttpStatus.CONFLICT)
                .body(error);
    }

    @ExceptionHandler(DataAccessException.class)
    public ResponseEntity<ApiError> handleDatabaseException(
            DataAccessException ex,
            HttpServletRequest request) {

        ApiError error = new ApiError(
            LocalDateTime.now(),
            500,
            "DATABASE_ERROR",
            "A database operation failed",
            request.getRequestURI()
        );

        return ResponseEntity
                .status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(error);
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ApiError> handleUnexpectedException(
            Exception ex,
            HttpServletRequest request) {

        ApiError error = new ApiError(
            LocalDateTime.now(),
            500,
            "INTERNAL_SERVER_ERROR",
            "An unexpected error occurred",
            request.getRequestURI()
        );

        return ResponseEntity
                .status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(error);
    }
}


Production improvement: The handlers above illustrate the architecture. In real production code, unexpected exceptions and database failures should be logged with the original exception (ex) using a logger, but the internal database message, SQL, table names, credentials, or stack traces should not be sent to the client.

11. Example HTTP error responses
Account not found

{
  "timestamp": "2026-11-23T12:10:30",
  "status": 404,
  "error": "NOT_FOUND",
  "message": "Account not found with ID: 999",
  "path": "/api/accounts/999"
}



Duplicate account

{
  "timestamp": "2026-11-23T12:10:30",
  "status": 409,
  "error": "CONFLICT",
  "message": "Account number already exists",
  "path": "/api/accounts"
}



Database failure

{
  "timestamp": "2026-11-23T12:10:30",
  "status": 500,
  "error": "DATABASE_ERROR",
  "message": "A database operation failed",
  "path": "/api/accounts"
}




12. Logging exceptions correctly
Never rely only on System.out.println() in production.

Use SLF4J:
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Service
public class AccountService {

    private static final Logger log =
            LoggerFactory.getLogger(AccountService.class);

    public Account getAccount(Long accountId) {

        try {
            return accountRepository.findById(accountId);

        } catch (EmptyResultDataAccessException e) {

            log.warn("Account not found. accountId={}", accountId);

            throw new AccountNotFoundException(accountId);
        }
    }
}


For unexpected database errors:
log.error("Database operation failed", ex);


This logs the stack trace for developers while allowing your API to return a safe message.

13. Retry: when should we retry database failures?
Not every database exception should be retried.
Error	Retry?
Invalid SQL	❌ No
Duplicate key	❌ No
Account not found	❌ No
Invalid data	❌ No
Temporary connection failure	⚠️ Possibly
Deadlock	⚠️ Possibly
Transaction serialization failure	⚠️ Possibly
Query timeout	⚠️ Depends

A production retry strategy should use:
- Limited retry attempts
- Exponential backoff
- Only retryable failures
- Transaction awareness
- Idempotent operations
For example, retrying a payment or deposit without idempotency protection can cause duplicate financial operations.

14. Production error-handling rules
Follow these rules in your JDBC/Spring applications
- Never swallow database exceptions.
- Don't expose SQL or stack traces to API clients.
- Keep business exceptions separate from database exceptions.
- Use @Transactional at the service layer.
- Let Spring translate SQL exceptions.
- Log unexpected errors with their stack traces.
- Retry only transient failures.
- Use database constraints for data integrity.
- Don't use exceptions for normal control flow.
- Make financial operations idempotent when retries are possible.

15. Interview questions
Q1. What is exception translation in Spring JDBC?

Spring translates vendor-specific SQLException instances into a consistent DataAccessException hierarchy.

Q2. Why is DataAccessException unchecked?

It reduces boilerplate and allows exceptions to propagate without forcing every method to declare or catch database exceptions.

Q3. What is DuplicateKeyException?

A Spring exception generally indicating that an insert/update violated a uniqueness constraint.

Q4. What happens when queryForObject() returns zero rows?

It typically throws EmptyResultDataAccessException.

Q5. Where should database exceptions be handled?
- Repository: database access.
- Service: business-level translation.
- Global handler: HTTP error conversion.
Q6. Why shouldn't we return raw SQL exception messages to clients?

They can expose internal implementation details, SQL, schema names, and potentially sensitive information.

Q7. Does @Transactional rollback on checked exceptions?

Not by default. By default, Spring rolls back for unchecked exceptions and errors. Use rollbackFor when checked exceptions should trigger rollback.

Q8. Should we retry every DataAccessException?

No. Many errors are permanent, such as invalid SQL, duplicate keys, and data integrity violations.

Revision summary
Concept	What to remember
DataAccessException	Spring's database exception base class
Exception translation	JDBC exceptions → Spring exceptions
DuplicateKeyException	Duplicate/unique constraint violation
EmptyResultDataAccessException	No result returned
Business exceptions	Application-level meaning
@RestControllerAdvice	Centralized API exception handling
@Transactional	Transaction boundary and rollback
SLF4J	Production logging
Retry	Only transient failures with safe retry logic

Next topic
B2.6 — Spring JDBC Transactions: @Transactional, Propagation, Isolation & Rollback

This will connect your existing JDBC transaction knowledge to how Spring manages transactions in real applications.