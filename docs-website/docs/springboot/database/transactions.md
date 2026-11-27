# B2.6 — Spring JDBC Transactions: @Transactional, Propagation, Isolation & Rollback
Before implementing transactions in our banking project, let's understand how Spring manages database transactions internally and how to use them safely.

We'll use PostgreSQL and our existing accounts and transactions tables for practical examples.
1. What is a database transaction?
A transaction is a group of database operations treated as one logical unit.

Imagine a customer deposits ₹1,000.

Two operations must happen:
1. Increase the account balance by ₹1,000.
2. Insert a deposit record into the transactions table.
Both operations must succeed together.
Successful transaction
Balance: ₹5,000 → ₹6,000
Deposit record: ₹1,000 inserted
COMMIT
Both changes are permanently saved.

Failed transaction
Balance update: ₹5,000 → ₹6,000
Transaction record insertion fails
ROLLBACK
Balance returns to ₹5,000. No deposit record is saved.

This is the purpose of transaction management.
2. ACID properties
Transactions follow four important properties.
Property	Meaning	Banking example
Atomicity	All operations succeed or fail together	Balance and history update together
Consistency	Database rules remain valid	Balance cannot become negative
Isolation	Concurrent transactions do not interfere incorrectly	Two withdrawals cannot overspend
Durability	Committed changes survive failures	Saved deposit remains recorded

These are the foundation of reliable database operations.
3. JDBC transactions vs Spring transactions
A. Traditional JDBC transaction
In plain JDBC, you manage the transaction manually:
Connection connection = dataSource.getConnection();

try {
    connection.setAutoCommit(false);

    // Operation 1
    updateAccountBalance(connection);

    // Operation 2
    insertTransactionHistory(connection);

    connection.commit();

} catch (Exception ex) {

    connection.rollback();

    throw new RuntimeException(ex);

} finally {
    connection.close();
}


Important methods:
- setAutoCommit(false) — disables automatic commit.
- commit() — saves all transaction changes.
- rollback() — reverses uncommitted changes.
- close() — releases the connection.
B. Spring transaction management
Spring simplifies this using @Transactional.
@Service
public class AccountService {

    @Transactional
    public void deposit(Long accountId, BigDecimal amount) {

        accountRepository.deposit(accountId, amount);

        transactionRepository.insertDeposit(
                accountId,
                amount
        );
    }
}


Spring manages the transaction lifecycle.

Conceptually:
Call deposit()
      |
      v
Spring starts transaction
      |
      v
Update account balance
      |
      v
Insert transaction record
      |
      v
Method completes successfully?
      |
   +--+--+
   |     |
  Yes    No
   |     |
 COMMIT ROLLBACK


Important: @Transactional works through Spring's transaction infrastructure. With Spring Boot JDBC auto-configuration, Spring normally configures a DataSourceTransactionManager (or its JDBC equivalent) for your DataSource.
4. How does @Transactional work internally?
When you call a Spring-managed service method annotated with @Transactional, Spring's transaction infrastructure typically uses an AOP proxy.

For example:
@Service
public class AccountService {

    @Transactional
    public void deposit(Long accountId, BigDecimal amount) {
        // Database operations
    }
}


The simplified execution flow is:
1. Your controller calls the Spring-managed AccountService proxy.
2. The proxy checks the transaction configuration.
3. Spring starts a transaction and obtains a database connection.
4. JdbcTemplate participates in that transaction.
5. The service method executes.
6. If the method succeeds, Spring commits.
7. If an applicable exception escapes the method, Spring rolls back.
Important: JdbcTemplate automatically participates
You do not need to manually call connection.setAutoCommit(false) when using Spring-managed transactions with JdbcTemplate.

Spring binds the transactional connection to the current execution thread. JdbcTemplate obtains and uses that connection through Spring's connection-management infrastructure.
Important proxy limitation
This works:
@Service
public class AccountService {

    public void processDeposit(Long id, BigDecimal amount) {
        deposit(id, amount);
    }

    @Transactional
    public void deposit(Long id, BigDecimal amount) {
        // Database operations
    }
}


But calling deposit() from processDeposit() on the same object is a self-invocation. It bypasses the Spring proxy, so the transaction annotation may not be applied.

The safest approach is to call transactional methods through another Spring-managed bean.

Also remember:
- The class and method must be eligible for Spring proxying.
- Use public service methods as the normal transaction entry points.
- Database operations must use the same transaction-managed DataSource to participate in the same JDBC transaction.

5. Rollback rules — When does Spring roll back?
By default, Spring rolls back when an unchecked exception escapes the transactional method:
- RuntimeException
- Error
By default, Spring does not roll back for checked exceptions such as IOException.
Example A: RuntimeException
@Transactional
public void deposit(Long accountId, BigDecimal amount) {

    accountRepository.deposit(accountId, amount);

    throw new RuntimeException("Something went wrong");
}


Result:
Balance update
     ↓
RuntimeException
     ↓
ROLLBACK


The balance update is rolled back.
Example B: Checked exception
@Transactional
public void deposit(Long accountId, BigDecimal amount)
        throws IOException {

    accountRepository.deposit(accountId, amount);

    throw new IOException("File processing failed");
}


By default, Spring commits the database transaction even though the checked exception escapes the method.

That distinction is extremely important.
Example C: rollbackFor
If you want rollback for checked exceptions, configure it explicitly:
@Transactional(rollbackFor = Exception.class)
public void deposit(Long accountId, BigDecimal amount)
        throws IOException {

    accountRepository.deposit(accountId, amount);

    throw new IOException("Processing failed");
}


Now the database transaction rolls back.
Example D: noRollbackFor
You can also explicitly exclude an exception from rollback:
@Transactional(noRollbackFor = IllegalArgumentException.class)
public void processAccount() {

    // Database operation

    throw new IllegalArgumentException("Warning condition");
}


If this exception escapes, Spring does not roll back solely because of it.

Use this carefully. If another rollback-triggering exception occurs, the transaction can still roll back.
Rollback summary
Exception	Default behavior
RuntimeException	Rollback
Error	Rollback
IOException	Commit
SQLException escaping directly	Checked exception; no rollback by default
Spring DataAccessException	Rollback

Notice that Spring JDBC usually translates SQLException into unchecked DataAccessException. That is why database failures normally trigger rollback.

Another important rule: If you catch an exception inside the transactional method and suppress it, Spring may see a successful method completion and commit. Don't swallow exceptions when the transaction must roll back.

6. Transaction Propagation
Propagation answers this question:

What should Spring do when a transactional method calls another transactional method?

Suppose:
@Transactional
public void transferMoney() {

    debitAccount();

    creditAccount();
}


Both methods participate in the transaction started by transferMoney() under the default propagation behavior.

Spring provides several propagation options.
A. REQUIRED — Default propagation
@Transactional(propagation = Propagation.REQUIRED)
public void transferMoney() {
    // Transaction logic
}


Behavior:
- If a transaction already exists, join it.
- Otherwise, create a new transaction.
Example:
transferMoney()
      |
      v
Transaction T1 starts
      |
      +---- debitAccount() joins T1
      |
      +---- creditAccount() joins T1
      |
      v
Commit T1


If either operation fails with a rollback-triggering exception, the whole transaction rolls back.
Most commonly used
B. REQUIRES_NEW
Always starts an independent transaction.
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void saveAuditLog() {
    // Independent database operation
}


If another transaction exists, Spring suspends it and starts a new one.
Outer transaction T1
       |
       v
Suspend T1
       |
       v
Start T2
       |
       v
Commit T2
       |
       v
Resume T1


Example use case: saving an audit record independently of a business transaction.

Be careful: if T2 commits and T1 later rolls back, the audit record remains.

Also, REQUIRES_NEW needs another database connection while the outer transaction is suspended. Under load, this can cause connection-pool pressure.
C. SUPPORTS
@Transactional(propagation = Propagation.SUPPORTS)
public void findAccount() {
}


- If a transaction exists, join it.
- Otherwise, execute without a transaction.
Useful when a method can work either inside or outside a transaction.
D. NOT_SUPPORTED
@Transactional(propagation = Propagation.NOT_SUPPORTED)
public void generateReport() {
}


- Suspends an existing transaction.
- Executes without a transaction.
- Resumes the previous transaction afterward.
Useful for operations that should not hold a database transaction open.
E. MANDATORY
@Transactional(propagation = Propagation.MANDATORY)
public void performInternalOperation() {
}


- Requires an existing transaction.
- Throws an exception if none exists.
Useful when a method must only be called as part of a larger transactional operation.
F. NEVER
@Transactional(propagation = Propagation.NEVER)
public void performNonTransactionalOperation() {
}


- Requires that no transaction exists.
- Throws an exception if one is already active.
G. NESTED
@Transactional(propagation = Propagation.NESTED)
public void processOptionalOperation() {
}


Can create a nested transaction using a savepoint inside an existing transaction.

If the nested operation fails and its failure is handled appropriately, Spring can roll back to the savepoint while allowing the outer transaction to continue.

Support depends on the transaction manager and database. With JDBC, this commonly uses savepoints.
Propagation comparison
Propagation	Existing transaction?	No existing transaction?
REQUIRED	Join	Create
REQUIRES_NEW	Suspend and create	Create
SUPPORTS	Join	No transaction
NOT_SUPPORTED	Suspend	No transaction
MANDATORY	Join	Throw exception
NEVER	Throw exception	No transaction
NESTED	Nested savepoint, if supported	Create

Practical recommendation: Start with REQUIRED. Learn REQUIRES_NEW and NESTED when you have a real use case for independent or savepoint-based work.

7. Transaction Isolation Levels
Isolation controls how much one transaction can see changes made by other concurrent transactions.

Imagine two bank employees are processing operations on the same account simultaneously.

Isolation levels help control what each transaction can observe.
Three important concurrency problems
1. Dirty Read

Transaction T1 reads data changed by T2 before T2 commits.

If T2 rolls back, T1 has read data that was never committed.

2. Non-repeatable Read

T1 reads an account balance. T2 updates and commits that balance. T1 reads the same account again and gets a different value.

3. Phantom Read

T1 runs a query that returns 10 accounts. T2 inserts another matching account and commits. T1 repeats the query and gets 11 accounts.
Spring isolation levels
You configure isolation using @Transactional(isolation = ...).
A. READ_UNCOMMITTED
@Transactional(isolation = Isolation.READ_UNCOMMITTED)
public void readAccounts() {
}


Allows the weakest isolation according to the SQL standard.

PostgreSQL note: PostgreSQL treats READ_UNCOMMITTED as READ_COMMITTED. It does not permit dirty reads.
B. READ_COMMITTED
@Transactional(isolation = Isolation.READ_COMMITTED)
public void readAccounts() {
}


Each query sees only committed data as of the beginning of that query.

This is PostgreSQL's default isolation level.
C. REPEATABLE_READ
@Transactional(isolation = Isolation.REPEATABLE_READ)
public void readAccounts() {
}


PostgreSQL provides a consistent snapshot for the transaction. Repeated reads see the same committed database snapshot.

PostgreSQL's implementation also prevents phantom reads under this isolation level, although serialization anomalies can still occur.
D. SERIALIZABLE
@Transactional(isolation = Isolation.SERIALIZABLE)
public void transferMoney() {
}


Provides the strongest isolation.

PostgreSQL attempts to make concurrent transactions behave as if they ran sequentially.

However, transactions can fail with serialization errors. Your application must be prepared to retry the entire transaction when appropriate.
Isolation comparison
Isolation	Dirty reads	Non-repeatable reads	Phantom reads
READ_UNCOMMITTED	Standard allows	Possible	Possible
READ_COMMITTED	Prevented	Possible	Possible
REPEATABLE_READ	Prevented	Prevented	Prevented in PostgreSQL
SERIALIZABLE	Prevented	Prevented	Prevented

The table describes PostgreSQL behavior where relevant; SQL-standard behavior can differ between database engines.

Practical recommendation: Use PostgreSQL's default READ_COMMITTED for many ordinary CRUD operations. Choose stronger isolation only when the business rules require it.

8. Real banking example — Atomic money transfer
Suppose:
- Account A has ₹10,000.
- Account B has ₹5,000.
- Transfer amount is ₹2,000.
The following operations must succeed together:
1. Debit ₹2,000 from Account A.
2. Credit ₹2,000 to Account B.
3. Insert a withdrawal record.
4. Insert a deposit record.
If any operation fails, all four must roll back.

Here's the service-level transaction boundary:
@Service
public class TransferService {

    private final AccountRepository accountRepository;
    private final TransactionRepository transactionRepository;

    public TransferService(
            AccountRepository accountRepository,
            TransactionRepository transactionRepository) {

        this.accountRepository = accountRepository;
        this.transactionRepository = transactionRepository;
    }

    @Transactional(rollbackFor = Exception.class)
    public void transfer(
            Long fromAccountId,
            Long toAccountId,
            BigDecimal amount) throws Exception {

        if (fromAccountId.equals(toAccountId)) {
            throw new IllegalArgumentException(
                    "Source and destination accounts must be different"
            );
        }

        if (amount == null || amount.compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException(
                    "Transfer amount must be positive"
            );
        }

        // 1. Debit source account
        int debited = accountRepository.withdraw(
                fromAccountId,
                amount
        );

        if (debited != 1) {
            throw new InsufficientBalanceException();
        }

        // 2. Credit destination account
        int credited = accountRepository.deposit(
                toAccountId,
                amount
        );

        if (credited != 1) {
            throw new AccountNotFoundException(toAccountId);
        }

        // 3. Insert withdrawal history
        transactionRepository.insertWithdrawal(
                fromAccountId,
                amount
        );

        // 4. Insert deposit history
        transactionRepository.insertDeposit(
                toAccountId,
                amount
        );
    }
}


This is an illustration of the transaction boundary. The TransactionRepository methods will be implemented when we build transaction history in our project.

Notice that withdraw() and deposit() must return their affected-row counts. Our earlier repository methods already follow that pattern.
What happens if the destination account doesn't exist?
Start transaction
       |
       v
Debit source account
       |
       v
Destination update fails
       |
       v
Throw AccountNotFoundException
       |
       v
ROLLBACK
       |
       v
Source account gets its money back


This is atomicity in practice.

Important: The database operations must participate in the same Spring-managed transaction. Do not manually open unrelated JDBC connections inside these repository methods.

9. readOnly, timeout and other transaction attributes
Spring supports additional transaction settings.
Read-only transaction
@Transactional(readOnly = true)
public List<Account> getAllAccounts() {
    return accountRepository.findAll();
}


Useful for read-only service operations.

readOnly = true is generally a transaction hint. Do not treat it as a universal guarantee that database writes are impossible.
Transaction timeout
@Transactional(timeout = 10)
public void processTransfer() {
    // Transaction should complete within 10 seconds
}


The timeout is expressed in seconds.

It is not a replacement for JDBC query timeouts or database statement timeouts.
Complete configuration example
@Transactional(
    propagation = Propagation.REQUIRED,
    isolation = Isolation.READ_COMMITTED,
    rollbackFor = Exception.class,
    timeout = 10
)
public void transferMoney() {
    // Business operations
}


You don't need to specify every attribute every time. Defaults are often sufficient.

10. Important production considerations
These are particularly important in banking applications.

1. Never put transaction boundaries in controllers.

Prefer service-level transactions because they define the complete business operation.

2. Don't keep transactions open unnecessarily.

Avoid external HTTP requests, long-running calculations, or user interactions inside a database transaction.

3. Handle concurrent withdrawals safely.

Our repository already uses conditional SQL:
UPDATE accounts
SET balance = balance - ?
WHERE account_id = ?
  AND status = 'ACTIVE'
  AND balance >= ?;


This is much safer than reading a balance into Java, checking it, and then blindly updating it.

4. Understand transaction rollback and exception handling.

If a failure must cause rollback, let an appropriate exception escape the transactional method or explicitly mark the transaction rollback-only.

5. Be careful with retries.

A serialization failure or deadlock may require retrying the entire transaction, not just the last SQL statement. Retrying should be limited and use a deliberate strategy.

6. Don't assume transactions automatically protect external side effects.

A database rollback cannot undo an email, HTTP request, or message already sent to another system. Patterns such as the transactional outbox help address this problem.

11. Quick revision — B2.6
Concept	Remember
@Transactional	Spring-managed transaction boundary
commit	Save changes
rollback	Undo uncommitted changes
rollbackFor	Configure additional rollback exceptions
REQUIRED	Join existing or create new
REQUIRES_NEW	Suspend existing and create independent
NESTED	Savepoint-based nested work, if supported
READ_COMMITTED	PostgreSQL default
REPEATABLE_READ	Stable transaction snapshot
SERIALIZABLE	Strongest isolation
readOnly	Read-only transaction hint
timeout	Transaction time limit

12. Test your understanding
B2.6 knowledge check
0/5 answered

1. What happens when a RuntimeException escapes a @Transactional method?

Transaction commits

Transaction rolls back

Connection pool shuts down



2. Which propagation is the default?

REQUIRES_NEW

NESTED

REQUIRED



3. What is PostgreSQL's default isolation level?

READ_COMMITTED

SERIALIZABLE

READ_UNCOMMITTED



4. What does REQUIRES_NEW do?

Always joins the outer transaction

Suspends the outer transaction and starts an independent one

Disables rollback



5. How do you request rollback for checked exceptions?

noRollbackFor

readOnly

rollbackFor


Check answers

B2.6 is now covered. The next step is to apply these concepts to our Banking Management API: implement atomic deposits, withdrawals, transfers and transaction-history insertion using @Transactional, with practical rollback testing.