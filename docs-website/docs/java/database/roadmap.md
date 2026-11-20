# 5. Your learning roadmap — Java first, Spring Boot second
This is the roadmap I recommend following. We will not mix Java JDBC topics with Spring abstractions prematurely.


## PART A — CORE JAVA DATABASE PROGRAMMING
Goal: Build a database application without Spring Boot.

### Phase A1 — JDBC Fundamentals
Already covered
- [x] What is JDBC?
- [x] JDBC architecture
- [x] JDBC API
- [x] JDBC Driver
- [x] PostgreSQL JDBC Driver
- [x] JDBC URL
- [x] Connection
- [x] Statement
- [x] PreparedStatement
- [x] CallableStatement
- [x] ResultSet
- [x] executeQuery()
- [x] executeUpdate()
- [x] execute()
- [x] Try-with-resources
- [x] SQL parameter binding

### Phase A2 — JDBC Project Architecture
Already covered
- [x] Database configuration
- [x] Connection management
- [x] Model classes
- [x] DAO pattern
- [x] Service layer
- [x] Database utility
- [x] Separation of concerns
- [x] CRUD operations
- [x] Repository/DAO responsibility

```
Suggested structure:
jdbc-project/
│
├── config/
│   └── DatabaseConfig.java
│
├── connection/
│   └── DatabaseConnection.java
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
│   └── DatabaseException.java
│
└── Main.java
```


### Phase A3 — JDBC Connection Management
Already covered
- [x] DriverManager
- [x] DataSource
- [x] Database connection lifecycle
- [x] Connection creation
- [x] Connection closing
- [x] Why connection creation is expensive
- [x] Connection reuse
- [x] Connection Pooling
- [x] HikariCP
- [x] Maximum pool size
- [x] Minimum idle
- [x] Connection timeout
- [x] Idle timeout
- [x] Max lifetime
- [x] Pool exhaustion
- [x] Connection leak concept
Remaining practical topics
- [ ] Connection leak detection
- [ ] Pool monitoring
- [ ] Pool sizing
- [ ] What happens when connections become invalid
- [ ] Connection validation
- [ ] Database outage behavior

### Phase A4 — JDBC Transactions
Already covered
- [x] Auto-commit
- [x] setAutoCommit(false)
- [x] Commit
- [x] Rollback
- [x] Transaction boundaries
- [x] ACID
- [x] Transactional operations
- [x] Money transfer example

### Phase A5 — Database Isolation
Already covered
- [x] Transaction isolation
- [x] READ COMMITTED
- [x] REPEATABLE READ
- [x] SERIALIZABLE
- [x] Dirty read
- [x] Non-repeatable read
- [x] Phantom read
- [x] PostgreSQL isolation behavior
- [x] JDBC setTransactionIsolation()
- [x] Transaction snapshots

### Phase A6 — Database Concurrency & Locking
Already covered
- [x] Concurrency
- [x] Race condition
- [x] Lost update
- [x] Atomic SQL
- [x] Optimistic locking
- [x] Pessimistic locking
- [x] Version column
- [x] SELECT FOR UPDATE
- [x] Row locks
- [x] Table locks
- [x] Advisory locks
- [x] Deadlocks
- [x] Deadlock prevention
- [x] Retry strategy
- [x] Serialization failures

### Phase A7 — JDBC Batch Processing & Performance
Next topic
- [ ] Batch insert
- [ ] Batch update
- [ ] addBatch()
- [ ] executeBatch()
- [ ] Batch size
- [ ] Batch processing with PreparedStatement
- [ ] Database round trips
- [ ] JDBC performance
- [ ] Bulk operations
- [ ] Processing large datasets
- [ ] fetchSize
- [ ] ResultSet streaming
- [ ] Memory management
- [ ] N+1 problem introduction
- [ ] Performance benchmarking

### Phase A8 — JDBC Exception Handling
- [ ] SQLException
- [ ] SQLState
- [ ] Vendor error codes
- [ ] Exception chaining
- [ ] Constraint violation
- [ ] Unique constraint violations
- [ ] Foreign key violations
- [ ] Database connection failures
- [ ] Query timeout
- [ ] Deadlock exception
- [ ] Serialization exception
- [ ] Exception translation in your Java application
- [ ] Custom database exceptions

Example:
```
try {
    // JDBC operation
} catch (SQLException ex) {
    System.out.println(ex.getSQLState());
    System.out.println(ex.getErrorCode());
    throw new DatabaseException("Database operation failed", ex);
}
```


### Phase A9 — JDBC Data Mapping
- [ ] ResultSet → Java Object
- [ ] RowMapper concept manually
- [ ] Object mapping
- [ ] Primitive vs wrapper types
- [ ] NULL handling
- [ ] SQL NULL vs Java null
- [ ] getObject()
- [ ] Date mapping
- [ ] Timestamp mapping
- [ ] BigDecimal mapping
- [ ] UUID mapping
- [ ] Enum mapping
- [ ] JSON/JSONB mapping
- [ ] PostgreSQL-specific data types

### Phase A10 — SQL Parameterization & Security
- [ ] SQL injection
- [ ] PreparedStatement security
- [ ] Parameter binding
- [ ] Dynamic SQL
- [ ] Dynamic table names
- [ ] Dynamic column names
- [ ] Whitelisting
- [ ] Least privilege
- [ ] Database credentials
- [ ] Secrets management
- [ ] Avoiding SQL string concatenation

### Phase A11 — Database Schema & Java Integration
This ### Phase is about how Java applications work with schema structures.
- [ ] Primary keys
- [ ] Foreign keys
- [ ] Unique constraints
- [ ] Check constraints
- [ ] Defaults
- [ ] Nullable columns
- [ ] Schema
- [ ] Table
- [ ] Column
- [ ] Data types
- [ ] Java ↔ SQL type mapping
- [ ] Database metadata
- [ ] JDBC DatabaseMetaData
- [ ] Table metadata retrieval
- [ ] Column metadata retrieval

### Phase A12 — JDBC Advanced
- [ ] DatabaseMetaData
- [ ] ResultSetMetaData
- [ ] CallableStatement
- [ ] Stored procedures
- [ ] Stored functions
- [ ] Batch processing
- [ ] Savepoints
- [ ] Generated keys
- [ ] Statement timeout
- [ ] PreparedStatement caching concepts
- [ ] Scrollable ResultSet
- [ ] Updatable ResultSet
- [ ] Transaction savepoints


Example generated keys:

```
PreparedStatement statement = connection.prepareStatement(
    "INSERT INTO employees(name) VALUES (?)",
    Statement.RETURN_GENERATED_KEYS
);
```


### Phase A13 — Database Design from Java Perspective
- [ ] DAO pattern
- [ ] Repository pattern
- [ ] Service pattern
- [ ] DTO
- [ ] Entity/model separation
- [ ] Data access abstraction
- [ ] Dependency injection concepts manually
- [ ] Interface-based DAO
- [ ] Unit testing DAO
- [ ] Integration testing
- [ ] Mocking database dependencies
- [ ] Testcontainers introduction

## PART B — SPRING BOOT DATABASE DEVELOPMENT
Goal: Understand what Spring simplifies and how production Spring Boot applications interact with databases.

We begin this only after completing Part A.

### Phase B1 — Spring Database Integration Fundamentals
- [ ] What Spring provides
- [ ] Spring IoC
- [ ] Dependency injection
- [ ] Beans
- [ ] ApplicationContext
- [ ] DataSource bean
- [ ] Configuration properties
- [ ] Environment variables
- [ ] Profiles
- [ ] Database configuration
- [ ] Auto-configuration
- [ ] Spring Boot JDBC starter


Understand:
```
Spring Boot
    ↓
Auto Configuration
    ↓
DataSource Bean
    ↓
HikariCP
    ↓
PostgreSQL JDBC Driver
    ↓
PostgreSQL
```


### Phase B2 — Spring JDBC
Important before JPA
- [ ] What Spring JDBC solves
- [ ] JDBC vs Spring JDBC
- [ ] JdbcTemplate
- [ ] DataSource
- [ ] query()
- [ ] queryForObject()
- [ ] update()
- [ ] execute()
- [ ] RowMapper
- [ ] ResultSetExtractor
- [ ] PreparedStatementSetter
- [ ] NamedParameterJdbcTemplate
- [ ] Named parameters
- [ ] Batch operations
- [ ] BatchPreparedStatementSetter
- [ ] SQL exception translation
- [ ] DataAccessException
- [ ] DAO with JdbcTemplate
- [ ] Service layer with JdbcTemplate
- [ ] Transaction integration


Example:

```
@Repository
public class EmployeeRepository {

    private final JdbcTemplate jdbcTemplate;

    public EmployeeRepository(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public Employee findById(Long id) {

        String sql = """
            SELECT id, name, salary
            FROM employees
            WHERE id = ?
            """;

        return jdbcTemplate.queryForObject(
            sql,
            (rs, rowNum) -> new Employee(
                rs.getLong("id"),
                rs.getString("name"),
                rs.getBigDecimal("salary")
            ),
            id
        );
    }
}
```

#### When to use Spring JDBC

Use it when:
- You want full SQL control.
- You have complex queries.
- You work with dynamic SQL.
- You need bulk processing.
- You want to optimize PostgreSQL queries.
- You need metadata queries.
- You don't want ORM behavior.

### Phase B3 — Spring Transaction Management
This is where the Java transaction concepts become Spring concepts.
- [ ] What Spring transactions solve
- [ ] TransactionManager
- [ ] PlatformTransactionManager
- [ ] DataSourceTransactionManager
- [ ] @Transactional
- [ ] Transaction propagation
- [ ] REQUIRED
- [ ] REQUIRES_NEW
- [ ] SUPPORTS
- [ ] NOT_SUPPORTED
- [ ] MANDATORY
- [ ] NEVER
- [ ] NESTED
- [ ] Rollback rules
- [ ] rollbackFor
- [ ] noRollbackFor
- [ ] RuntimeException rollback
- [ ] Checked exception rollback
- [ ] Transaction boundaries
- [ ] Transaction proxy
- [ ] Self-invocation problem
- [ ] Transaction isolation in Spring
- [ ] Transaction timeout
- [ ] Read-only transactions
- [ ] Transaction synchronization


#### Example:
```
@Transactional
public void transferMoney() {
    // Debit
    // Credit
}
```

Advanced:
```
@Transactional(
    isolation = Isolation.REPEATABLE_READ,
    propagation = Propagation.REQUIRED,
    timeout = 10
)
public void process() {
}
```


### Phase B4 — JPA Fundamentals
Major ### Phase
- [ ] What ORM is
- [ ] What JPA solves
- [ ] JDBC vs ORM
- [ ] JPA specification
- [ ] Hibernate
- [ ] Entity
- [ ] @Entity
- [ ] @Table
- [ ] @Id
- [ ] @GeneratedValue
- [ ] EntityManager
- [ ] Persistence Context
- [ ] Entity lifecycle
- [ ] Transient
- [ ] Managed
- [ ] Detached
- [ ] Removed
- [ ] Persist
- [ ] Merge
- [ ] Remove
- [ ] Flush
- [ ] Clear
- [ ] Refresh

### Phase B5 — Hibernate Fundamentals
- [ ] Hibernate architecture
- [ ] Session
- [ ] SessionFactory
- [ ] EntityManager
- [ ] Persistence Context
- [ ] Hibernate SQL generation
- [ ] SQL logging
- [ ] Hibernate configuration
- [ ] Hibernate dialect
- [ ] Hibernate caching
- [ ] Hibernate dirty checking
- [ ] Hibernate flush behavior
- [ ] First-level cache
- [ ] Second-level cache
- [ ] Query cache concepts

### Phase B6 — Entity Mapping
- [ ] Basic mapping
- [ ] Java types → SQL types
- [ ] String
- [ ] Integer
- [ ] Long
- [ ] Boolean
- [ ] BigDecimal
- [ ] LocalDate
- [ ] LocalDateTime
- [ ] Instant
- [ ] UUID
- [ ] Enum
- [ ] @Column
- [ ] nullable
- [ ] unique
- [ ] length
- [ ] precision
- [ ] scale
- [ ] @Transient
- [ ] @Temporal concepts
- [ ] Custom converters
- [ ] AttributeConverter

### Phase B7 — Primary Keys & ID Generation
- [ ] Identity
- [ ] Sequence
- [ ] Table
- [ ] UUID
- [ ] @GeneratedValue
- [ ] Generation strategies
- [ ] PostgreSQL sequence
- [ ] UUID generation
- [ ] Custom ID generator concepts
- [ ] Composite keys
- [ ] @EmbeddedId
- [ ] @IdClass

### Phase B8 — Relationships
Critical
- [ ] One-to-one
- [ ] One-to-many
- [ ] Many-to-one
- [ ] Many-to-many
- [ ] @JoinColumn
- [ ] @JoinTable
- [ ] Cascade
- [ ] CascadeType
- [ ] FetchType
- [ ] LAZY
- [ ] EAGER
- [ ] Ownership
- [ ] Bidirectional relationship
- [ ] Unidirectional relationship
- [ ] mappedBy
- [ ] Orphan removal
- [ ] Foreign key mapping

### Phase B9 — Hibernate Fetching & Performance
- [ ] Lazy loading
- [ ] Eager loading
- [ ] Proxy
- [ ] N+1 problem
- [ ] Fetch join
- [ ] EntityGraph
- [ ] Batch fetching
- [ ] @BatchSize concepts
- [ ] Pagination
- [ ] Query performance
- [ ] SQL monitoring
- [ ] Hibernate statistics
- [ ] Database indexes
- [ ] Avoid unnecessary entity loading

### Phase B10 — JPQL & Querying
- [ ] JPQL
- [ ] Entity queries
- [ ] Select
- [ ] Where
- [ ] Join
- [ ] Group By
- [ ] Order By
- [ ] Aggregate functions
- [ ] Subqueries
- [ ] Named queries
- [ ] Parameters
- [ ] Pagination
- [ ] Sorting

### Phase B11 — Spring Data JPA
- [ ] JpaRepository
- [ ] CrudRepository
- [ ] Repository
- [ ] Repository methods
- [ ] save()
- [ ] findById()
- [ ] findAll()
- [ ] deleteById()
- [ ] existsById()
- [ ] Derived query methods
- [ ] findByName()
- [ ] findByStatusAndType()
- [ ] findByCreatedAtBetween()
- [ ] Query annotation
- [ ] Native queries
- [ ] Specifications
- [ ] Criteria API
- [ ] Pagination
- [ ] Sorting
- [ ] Projections
- [ ] DTO projections

### Phase B12 — Spring Data JPA Advanced
- [ ] Custom repository implementations
- [ ] Specifications
- [ ] Criteria API
- [ ] Dynamic queries
- [ ] Specification composition
- [ ] Projections
- [ ] DTO queries
- [ ] Native SQL
- [ ] Entity graphs
- [ ] Pagination optimization
- [ ] Slice vs Page
- [ ] Repository customization

### Phase B13 — JPA Transactions & Concurrency
This revisits what you already learned in Java and applies it to Hibernate.
- [ ] @Transactional
- [ ] Persistence context
- [ ] Dirty checking
- [ ] Flush
- [ ] Commit
- [ ] Rollback
- [ ] Optimistic locking
- [ ] @Version
- [ ] OptimisticLockException
- [ ] Pessimistic locking
- [ ] LockModeType
- [ ] PESSIMISTIC_READ
- [ ] PESSIMISTIC_WRITE
- [ ] Lock timeout
- [ ] Deadlocks
- [ ] Transaction isolation
- [ ] Retry strategies

### Phase B14 — Database Migrations
- [ ] Why migrations are necessary
- [ ] Flyway
- [ ] Liquibase
- [ ] Migration files
- [ ] Versioning
- [ ] Schema changes
- [ ] Rollback strategy
- [ ] Production migration
- [ ] Data migration
- [ ] Migration conflicts

### Phase B15 — Database Testing
- [ ] Unit testing repositories
- [ ] Integration testing
- [ ] @DataJdbcTest
- [ ] @DataJpaTest
- [ ] @SpringBootTest
- [ ] Test database
- [ ] H2 limitations
- [ ] PostgreSQL integration tests
- [ ] Testcontainers
- [ ] Transactional test
- [ ] Rollback tests
- [ ] Test isolation
- [ ] Concurrent database tests

### Phase B16 — Production Database Architecture
- [ ] Connection pool monitoring
- [ ] Database health checks
- [ ] Actuator
- [ ] Database metrics
- [ ] Query logging
- [ ] Slow query detection
- [ ] Database timeouts
- [ ] Connection failures
- [ ] Retry mechanisms
- [ ] Circuit breaker concepts
- [ ] Secrets management
- [ ] Database configuration per environment
- [ ] Dev/Test/Production profiles
- [ ] Read replicas
- [ ] Database backup
- [ ] Recovery
- [ ] Pool sizing
- [ ] Database capacity planning

## PART C — What you should learn before moving to Spring
Before Part B, I recommend finishing the following Java-only topics. These will make Spring Boot much easier to understand.
### Phase C1 — Core Java required for database applications
- [ ] Classes and objects
- [ ] Constructors
- [ ] Encapsulation
- [ ] Interfaces
- [ ] Abstract classes
- [ ] Inheritance
- [ ] Polymorphism
- [ ] Composition
- [ ] Generics
- [ ] Collections
  - List
  - Set
  - Map
  - Queue
- [ ] Exception handling
- [ ] Checked vs unchecked exceptions
- [ ] Custom exceptions
- [ ] Try-with-resources
- [ ] Functional interfaces
- [ ] Lambda expressions
- [ ] Streams
- [ ] Optional
- [ ] Records
- [ ] Enums
### Phase C2 — Java application architecture
- [ ] Layered architecture
- [ ] Controller concepts
- [ ] Service layer
- [ ] Repository / DAO
- [ ] DTO
- [ ] Model
- [ ] Configuration
- [ ] Utilities
- [ ] Interfaces for abstraction
- [ ] Dependency inversion
- [ ] SOLID principles
- [ ] Clean code
- [ ] Separation of concerns
### Phase C3 — Java concurrency
You have already started this topic, so continue separately.
- [ ] Thread
- [ ] Runnable
- [ ] Callable
- [ ] Future
- [ ] Thread lifecycle
- [ ] ExecutorService
- [ ] Thread pool
- [ ] ScheduledExecutorService
- [ ] CompletableFuture
- [ ] Synchronization
- [ ] synchronized
- [ ] Locks
- [ ] ReentrantLock
- [ ] Atomic variables
- [ ] Concurrent collections
- [ ] Thread safety
- [ ] Deadlock
- [ ] Race conditions
These are particularly important for database applications because database connections, transactions, concurrent requests, and background processing all involve concurrency.