# Phase 7.10 — JDBC Metadata & Database Introspection

This is a particularly important topic for your backend development because it allows Java to discover database structures dynamically instead of hardcoding table and column definitions.

You will use this concept in:
- Database administration tools
- Database migration tools
- JSON-to-DDL generators
- Data replication tools
- ETL pipelines
- Schema comparison tools
- Database monitoring applications
- Dynamic API generators
- Your Helyx data-processing projects

## 7.10.1 — What is Database Introspection?
Database introspection means examining a database programmatically to understand its structure.

Instead of manually writing:

```
String tableName = "employees";

String[] columns = {
    "id",
    "name",
    "email",
    "salary"
};
```

Java can ask the database:
- What tables exist?
- What columns exist?
- What are their data types?
- Which columns are primary keys?
- Which columns are nullable?
- What are their sizes?
- Which tables have foreign keys?
- Which indexes exist?

The database itself becomes the source of truth.
Database introspection workflow

```
PostgreSQL Database
Tables, columns, constraints, indexes

JDBC Metadata APIs
DatabaseMetaData / ResultSetMetaData

Java Introspection Logic
Read and interpret database structure

Dynamic Metadata Objects
Tables, columns, keys, relationships
```

## 7.10.2 — Two Important JDBC Metadata APIs
JDBC provides two important metadata interfaces.
### DatabaseMetaData
Database-level metadata

- Tables
- Schemas
- Primary keys
- Foreign keys
- Indexes
- Database information
- Driver information


### ResultSetMetaData
Query-level metadata

- Column names
- Column labels
- Column types
- Column count
- Nullable status
- Precision
- Scale



Difference

DatabaseMetaData - Inspect the database structure
ResultSetMetaData	- Inspect the columns of a query result


## 7.10.3 — DatabaseMetaData
DatabaseMetaData provides information about the database, its objects, and JDBC driver capabilities.

You obtain it from a JDBC connection:
```
DatabaseMetaData metaData = connection.getMetaData();
```

Example:
```
Connection connection = DriverManager.getConnection(url, user, password);

DatabaseMetaData metaData = connection.getMetaData();
```

Now Java can inspect the database.

Get database information

```
System.out.println(
        metaData.getDatabaseProductName()
);

System.out.println(
        metaData.getDatabaseProductVersion()
);

System.out.println(
        metaData.getDriverName()
);

System.out.println(
        metaData.getDriverVersion()
);
```

Example output:
```
PostgreSQL
17.5
PostgreSQL JDBC Driver
42.7.x
```

The actual output depends on the PostgreSQL server and JDBC driver installed.

#### Useful DatabaseMetaData methods
Method	Purpose

getDatabaseProductName() -	Database name
getDatabaseProductVersion() -	Database version
getDriverName() -	JDBC driver
getDriverVersion() -	Driver version
getURL() -	JDBC connection URL
getUserName() -	Connected database user
getCatalogs() -	Available catalogs
getSchemas() -	Available schemas
getTables() -	Discover tables and views
getColumns() -	Discover columns
getPrimaryKeys() -	Discover primary keys
getImportedKeys() -	Foreign keys referencing another table
getExportedKeys() -	Foreign keys referencing the selected table
getIndexInfo() -	Index information


## 7.10.4 — Discover Database Schemas
In PostgreSQL, a schema is a logical namespace containing database objects.

Example:
Database: helyx_db
```
Schemas:
├── public
├── analytics
├── staging
└── replication
```

You can discover schemas using JDBC.

```
DatabaseMetaData metaData = connection.getMetaData();

try (ResultSet rs = metaData.getSchemas()) {

    while (rs.next()) {

        String schemaName = rs.getString("TABLE_SCHEM");

        String catalogName = rs.getString("TABLE_CATALOG");

        System.out.println(
                "Schema: " + schemaName +
                ", Catalog: " + catalogName
        );
    }
}
```

#### What is the catalog?
A catalog represents a logical database grouping in the JDBC metadata model.

The precise meaning of catalog and schema varies by database.

In PostgreSQL:
- Database → database catalog-like scope
- Schema → namespace within the database
- Table → object inside a schema
Do not assume that catalog/schema meanings are identical across PostgreSQL, MySQL, Oracle, and SQL Server.

## 7.10.5 — Discover Tables Dynamically
This is one of the most useful operations.

Imagine your database contains:

```
public
├── employees
├── departments
├── projects
├── employee_projects
└── audit_logs
```

Instead of manually typing table names, you can discover them.

#### getTables()

```
DatabaseMetaData metaData = connection.getMetaData();

try (ResultSet rs = metaData.getTables(
        null,
        "public",
        "%",
        new String[]{"TABLE"}
)) {

    while (rs.next()) {

        String tableName = rs.getString("TABLE_NAME");

        String tableType = rs.getString("TABLE_TYPE");

        String schema = rs.getString("TABLE_SCHEM");

        System.out.println(
                schema + "." + tableName +
                " [" + tableType + "]"
        );
    }
}
```

Understanding the parameters

```
metaData.getTables(
    catalog,
    schemaPattern,
    tableNamePattern,
    types
);
```

Parameter	Meaning

```
catalog -	Catalog filter
schemaPattern -	Schema filter
tableNamePattern -	Table name filter
types -	TABLE, VIEW, etc.
```
For example:
```
metaData.getTables(
    null,
    "public",
    "%",
    new String[]{"TABLE", "VIEW"}
);
```

This retrieves tables and views from the public schema.

Important: TABLE vs VIEW
If you use:
```
new String[]{"TABLE"}
```

you are filtering for table types.

Depending on the driver and database, you may also encounter:
- VIEW
- MATERIALIZED VIEW
- FOREIGN TABLE
- SYSTEM TABLE
Metadata type reporting can vary by driver, so verify actual returned types when building a cross-database tool.

## 7.10.6 — Discover Columns Using getColumns()
Now suppose you discovered the employees table.

You want Java to discover the structure without hardcoding:
```
id INTEGER
name VARCHAR(100)
email VARCHAR(255)
salary NUMERIC(10,2)
joining_date DATE
```

Instead, Java will query the metadata.

### getColumns()

```
ResultSet rs = metaData.getColumns(
        null,
        "public",
        "employees",
        "%"
);
```

The method signature:

```
getColumns(
    String catalog,
    String schemaPattern,
    String tableNamePattern,
    String columnNamePattern
)
```

Complete example

```
DatabaseMetaData metaData = connection.getMetaData();

try (
    ResultSet rs = metaData.getColumns(
        null,
        "public",
        "employees",
        "%"
    )
) {

    while (rs.next()) {

        String columnName =
                rs.getString("COLUMN_NAME");

        int dataType =
                rs.getInt("DATA_TYPE");

        String typeName =
                rs.getString("TYPE_NAME");

        int columnSize =
                rs.getInt("COLUMN_SIZE");

        int decimalDigits =
                rs.getInt("DECIMAL_DIGITS");

        int nullable =
                rs.getInt("NULLABLE");

        int ordinalPosition =
                rs.getInt("ORDINAL_POSITION");

        System.out.println(
                columnName + " | " +
                dataType + " | " +
                typeName + " | " +
                columnSize + " | " +
                decimalDigits + " | " +
                nullable + " | " +
                ordinalPosition
        );
    }
}
```

Example output:

```
id | 4 | int4 | 32 | 0 | 0 | 1
name | 12 | varchar | 100 | 0 | 1 | 2
email | 12 | varchar | 255 | 0 | 1 | 3
salary | 2 | numeric | 10 | 2 | 1 | 4
joining_date | 91 | date | 0 | 0 | 1 | 5
```

Values shown are illustrative. Actual metadata values and column-size semantics depend on the PostgreSQL JDBC driver and SQL type.
Important getColumns() metadata fields
Field	Meaning

```
TABLE_CAT	Catalog/database
TABLE_SCHEM	Schema
TABLE_NAME	Table
COLUMN_NAME	Column name
DATA_TYPE	JDBC SQL type number
TYPE_NAME	Database-specific type name
COLUMN_SIZE	Column size or precision
DECIMAL_DIGITS	Scale
NULLABLE	Whether NULL is permitted
ORDINAL_POSITION	Column order
COLUMN_DEF	Default value, when available
IS_AUTOINCREMENT	Auto-increment indicator, when supported
IS_GENERATEDCOLUMN	Generated column indicator, when supported
```
This information is enough to start building a dynamic table metadata model.

## 7.10.7 — Understanding DATA_TYPE vs TYPE_NAME
This distinction is important.

Suppose PostgreSQL has:
```
id INTEGER
salary NUMERIC(10,2)
joining_date DATE
metadata JSONB
employee_id UUID
```

JDBC provides two type representations.
#### DATA_TYPE
```
int dataType = rs.getInt("DATA_TYPE");
```

This is generally a JDBC type code from java.sql.Types.

For example:

```
SQL Type	JDBC Type
INTEGER	Types.INTEGER
VARCHAR	Types.VARCHAR
NUMERIC	Types.NUMERIC
DATE	Types.DATE
TIMESTAMP	Types.TIMESTAMP
```

#### TYPE_NAME
```
String typeName = rs.getString("TYPE_NAME");
```

This is the database/driver type name.

Examples may include:

- int4
- varchar
- numeric
- date
- timestamp
- jsonb
- uuid


Why both matter
DATA_TYPE gives you a standardized JDBC type representation.

TYPE_NAME gives you database-specific information.

For a database-agnostic tool, use DATA_TYPE.

For PostgreSQL-specific tooling, also use TYPE_NAME.

For example, when building a JSON-to-DDL system or schema analyzer, you may need to distinguish PostgreSQL's:
```
json
jsonb
uuid
serial
bigserial
```

Some of these types may share JDBC type mappings, so TYPE_NAME can provide important additional detail.

## 7.10.8 — Discover Primary Keys
A primary key identifies a row uniquely.

Example:

```
CREATE TABLE employees (
    id INTEGER PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255)
);
```

Instead of hardcoding:
String primaryKey = "id";

Java can discover it.


### getPrimaryKeys()
```
ResultSet rs = metaData.getPrimaryKeys(
        null,
        "public",
        "employees"
);
```

Complete example:

```
try (
    ResultSet rs = metaData.getPrimaryKeys(
        null,
        "public",
        "employees"
    )
) {

    while (rs.next()) {

        String columnName =
                rs.getString("COLUMN_NAME");

        short keySequence =
                rs.getShort("KEY_SEQ");

        String constraintName =
                rs.getString("PK_NAME");

        System.out.println(
                "PK Column: " + columnName +
                ", Sequence: " + keySequence +
                ", Constraint: " + constraintName
        );
    }
}
```

Example output:
```
PK Column: id, Sequence: 1, Constraint: employees_pkey
```


### Composite Primary Keys
Suppose:

```
CREATE TABLE employee_projects (
    employee_id INTEGER,
    project_id INTEGER,
    assigned_date DATE,
    PRIMARY KEY (employee_id, project_id)
);
```

The primary key contains two columns:
- employee_id
- project_id


Metadata returns both.

The KEY_SEQ field tells you the order within the primary key.

```
Column	KEY_SEQ
employee_id	1
project_id	2
```

This is why your metadata model should not represent a primary key as only one string.

Instead, use something like:
```
List<String> primaryKeyColumns;
```


## 7.10.9 — Discover Foreign Keys
Foreign keys represent relationships between tables.

Example:
```
CREATE TABLE departments (
    id INTEGER PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE employees (
    id INTEGER PRIMARY KEY,
    name VARCHAR(100),
    department_id INTEGER,
    FOREIGN KEY (department_id)
        REFERENCES departments(id)
);
```

The relationship is:
#### departments

id — PRIMARY KEY


#### employees

department_id — FOREIGN KEY

employees.department_id → departments.id

### getImportedKeys()
Imported keys are foreign keys defined in the current table that reference another table.

```
ResultSet rs = metaData.getImportedKeys(
        null,
        "public",
        "employees"
);
```

Example:

```
try (
    ResultSet rs = metaData.getImportedKeys(
        null,
        "public",
        "employees"
    )
) {

    while (rs.next()) {

        String fkColumn =
                rs.getString("FKCOLUMN_NAME");

        String referencedTable =
                rs.getString("PKTABLE_NAME");

        String referencedColumn =
                rs.getString("PKCOLUMN_NAME");

        String constraintName =
                rs.getString("FK_NAME");

        System.out.println(
                fkColumn + " -> " +
                referencedTable + "." +
                referencedColumn +
                " [" + constraintName + "]"
        );
    }
}
```

Output:
```
department_id -> departments.id [employees_department_id_fkey]
```


Imported vs Exported Keys

```
Method	Meaning
getImportedKeys()	Foreign keys in this table referencing another table
getExportedKeys()	Foreign keys in other tables referencing this table
```

Suppose:
employees.department_id → departments.id


If you inspect employees:

- getImportedKeys() reveals its relationship to departments.

If you inspect departments:

- getExportedKeys() reveals that employees references it.

This distinction is important when building a database relationship graph.

## 7.10.10 — Discover Indexes
Indexes improve lookup performance, but metadata introspection can also discover them.

Suppose:

```
CREATE INDEX idx_employee_email
ON employees(email);
```

You can use:
```
metaData.getIndexInfo(
        null,
        "public",
        "employees",
        false,
        false
);
```

Signature:

```
getIndexInfo(
    catalog,
    schema,
    table,
    unique,
    approximate
)
```

Example:

```
try (
    ResultSet rs = metaData.getIndexInfo(
        null,
        "public",
        "employees",
        false,
        false
    )
) {

    while (rs.next()) {

        String indexName =
                rs.getString("INDEX_NAME");

        String columnName =
                rs.getString("COLUMN_NAME");

        boolean nonUnique =
                rs.getBoolean("NON_UNIQUE");

        System.out.println(
                "Index: " + indexName +
                ", Column: " + columnName +
                ", Non-Unique: " + nonUnique
        );
    }
}
```

Important metadata fields:

```
Field	Meaning
INDEX_NAME	Index name
COLUMN_NAME	Indexed column
NON_UNIQUE	Whether duplicates are allowed
ORDINAL_POSITION	Position within the index
INDEX_TYPE	Index type
```

Metadata can vary across drivers. PostgreSQL-specific index details such as partial indexes, expression indexes, INCLUDE columns, and index methods may require PostgreSQL system catalog queries.

## 7.10.11 — ResultSetMetaData
So far, we have used DatabaseMetaData to inspect database tables and columns.

Now let's learn ResultSetMetaData, which is used to inspect the result of a query.

For example:
```
SELECT
    id,
    name,
    salary,
    joining_date
FROM employees;
```

You might not know in advance:
- How many columns were returned?
- What is the column name?
- What is the SQL type?
- What is the Java class?
- What is the precision?
- Is it nullable?


ResultSetMetaData can answer these questions.

#### Getting ResultSetMetaData

```
PreparedStatement ps =
        connection.prepareStatement(
                "SELECT * FROM employees"
        );

ResultSet rs = ps.executeQuery();

ResultSetMetaData rsmd = rs.getMetaData();
```


Discover column count
```
int columnCount = rsmd.getColumnCount();

System.out.println("Columns: " + columnCount);
```


Discover columns dynamically
```
for (int i = 1; i <= rsmd.getColumnCount(); i++) {

    System.out.println(
            rsmd.getColumnName(i)
    );
}
```

Notice:
```
ResultSetMetaData column indexes start at 1, not 0.
```
Discover column properties

```
for (int i = 1; i <= rsmd.getColumnCount(); i++) {

    System.out.println("Column Name: "
            + rsmd.getColumnName(i));

    System.out.println("Column Label: "
            + rsmd.getColumnLabel(i));

    System.out.println("SQL Type: "
            + rsmd.getColumnType(i));

    System.out.println("Type Name: "
            + rsmd.getColumnTypeName(i));

    System.out.println("Java Class: "
            + rsmd.getColumnClassName(i));

    System.out.println("Precision: "
            + rsmd.getPrecision(i));

    System.out.println("Scale: "
            + rsmd.getScale(i));

    System.out.println("Nullable: "
            + rsmd.isNullable(i));

    System.out.println("----------------");
}
```

Important methods

```
Method	Description
getColumnCount()	Number of result columns
getColumnName(i)	Column name
getColumnLabel(i)	Column label/alias
getColumnType(i)	JDBC type code
getColumnTypeName(i)	Database-specific type name
getColumnClassName(i)	Java class associated with the column
getPrecision(i)	Precision
getScale(i)	Decimal scale
isNullable(i)	Nullable status
isAutoIncrement(i)	Auto-increment status
isSigned(i)	Signed numeric type
getTableName(i)	Source table name, when provided
getSchemaName(i)	Source schema name, when provided
```

## 7.10.12 — getColumnName() vs getColumnLabel()
This is a subtle but important concept.

Suppose you execute:

```
SELECT
    employee_id AS id,
    employee_name AS name
FROM employees;
```

You have renamed the output columns using aliases.

Now:
```
rsmd.getColumnName(1);
```

May return the underlying column name:
```
employee_id
```

While:
```
rsmd.getColumnLabel(1);
```

Returns the query label:
```
id
```

Recommendation
For dynamically processing query results, especially when SQL aliases are used, getColumnLabel() is often more appropriate because it reflects the column label visible in the query output.

Example:
```
SELECT
    salary * 12 AS annual_salary
FROM employees;
```

The result has a column label:
```
annual_salary
```

This is useful for generic data processing.

## 7.10.13 — DatabaseMetaData vs ResultSetMetaData — Practical Comparison
### DatabaseMetaData

```
DatabaseMetaData metadata =
    connection.getMetaData();
```

Used to discover the database structure.
- Tables
- Schemas
- Columns
- Primary keys
- Foreign keys
- Indexes

### ResultSetMetaData
```
ResultSetMetaData metadata =
    resultSet.getMetaData();
```

Used to discover the structure of the query result.
- Number of result columns
- Column labels
- SQL types
- Precision
- Scale
- Java classes

Example
Suppose:

```
SELECT
    e.name,
    d.name AS department_name,
    e.salary * 12 AS annual_salary
FROM employees e
JOIN departments d
    ON e.department_id = d.id;
```

DatabaseMetaData tells you what exists in the database.

ResultSetMetaData tells you what the query actually returned.

This is important because the query may:
- Select only a few columns
- Rename columns
- Calculate new columns
- Join multiple tables
- Return aggregate values
- Return expressions

## 7.10.14 — Build a Generic Database Introspection Utility
Now let's combine what you have learned.

We will create a reusable utility that can:
1. Connect to PostgreSQL.
2. Discover tables.
3. Discover columns.
4. Discover primary keys.
5. Discover foreign keys.
This is a small foundation for a metadata-driven tool.

### Step 1 — Create ColumnMetadata

```
package com.example.metadata;

public class ColumnMetadata {

    private String columnName;
    private String typeName;
    private int jdbcType;
    private int columnSize;
    private int decimalDigits;
    private boolean nullable;
    private int ordinalPosition;

    public ColumnMetadata(
            String columnName,
            String typeName,
            int jdbcType,
            int columnSize,
            int decimalDigits,
            boolean nullable,
            int ordinalPosition
    ) {
        this.columnName = columnName;
        this.typeName = typeName;
        this.jdbcType = jdbcType;
        this.columnSize = columnSize;
        this.decimalDigits = decimalDigits;
        this.nullable = nullable;
        this.ordinalPosition = ordinalPosition;
    }

    public String getColumnName() {
        return columnName;
    }

    public String getTypeName() {
        return typeName;
    }

    public int getJdbcType() {
        return jdbcType;
    }

    public int getColumnSize() {
        return columnSize;
    }

    public int getDecimalDigits() {
        return decimalDigits;
    }

    public boolean isNullable() {
        return nullable;
    }

    public int getOrdinalPosition() {
        return ordinalPosition;
    }
}
```


### Step 2 — Create TableMetadata

```
package com.example.metadata;

import java.util.ArrayList;
import java.util.List;

public class TableMetadata {

    private String tableName;
    private String schemaName;

    private List<ColumnMetadata> columns =
            new ArrayList<>();

    private List<String> primaryKeys =
            new ArrayList<>();

    public TableMetadata(String schemaName, String tableName) {
        this.schemaName = schemaName;
        this.tableName = tableName;
    }

    public String getTableName() {
        return tableName;
    }

    public String getSchemaName() {
        return schemaName;
    }

    public List<ColumnMetadata> getColumns() {
        return columns;
    }

    public List<String> getPrimaryKeys() {
        return primaryKeys;
    }

    public void addColumn(ColumnMetadata column) {
        columns.add(column);
    }

    public void addPrimaryKey(String columnName) {
        primaryKeys.add(columnName);
    }
}
```

Our metadata objects now represent database structures.
Metadata object structure

#### TableMetadata
employees

#### Columns
- id → INTEGER
- name → VARCHAR
- email → VARCHAR
- salary → NUMERIC
- joining_date → DATE


#### Primary Keys
- id

## 7.10.15 — Implement the Metadata Service
Let's create the core introspection logic.


### DatabaseIntrospectionService.java

```
package com.example.metadata;

import java.sql.Connection;
import java.sql.DatabaseMetaData;
import java.sql.ResultSet;
import java.sql.SQLException;

public class DatabaseIntrospectionService {

    private final Connection connection;

    public DatabaseIntrospectionService(Connection connection) {
        this.connection = connection;
    }

    public TableMetadata inspectTable(
            String schemaName,
            String tableName
    ) throws SQLException {

        DatabaseMetaData metaData =
                connection.getMetaData();

        TableMetadata tableMetadata =
                new TableMetadata(schemaName, tableName);

        inspectColumns(
                metaData,
                schemaName,
                tableName,
                tableMetadata
        );

        inspectPrimaryKeys(
                metaData,
                schemaName,
                tableName,
                tableMetadata
        );

        return tableMetadata;
    }

    private void inspectColumns(
            DatabaseMetaData metaData,
            String schemaName,
            String tableName,
            TableMetadata tableMetadata
    ) throws SQLException {

        try (
            ResultSet rs = metaData.getColumns(
                    null,
                    schemaName,
                    tableName,
                    "%"
            )
        ) {

            while (rs.next()) {

                String columnName =
                        rs.getString("COLUMN_NAME");

                String typeName =
                        rs.getString("TYPE_NAME");

                int jdbcType =
                        rs.getInt("DATA_TYPE");

                int columnSize =
                        rs.getInt("COLUMN_SIZE");

                int decimalDigits =
                        rs.getInt("DECIMAL_DIGITS");

                int nullableValue =
                        rs.getInt("NULLABLE");

                boolean nullable =
                        nullableValue !=
                        DatabaseMetaData.columnNoNulls;

                int ordinalPosition =
                        rs.getInt("ORDINAL_POSITION");

                ColumnMetadata column =
                        new ColumnMetadata(
                                columnName,
                                typeName,
                                jdbcType,
                                columnSize,
                                decimalDigits,
                                nullable,
                                ordinalPosition
                        );

                tableMetadata.addColumn(column);
            }
        }
    }

    private void inspectPrimaryKeys(
            DatabaseMetaData metaData,
            String schemaName,
            String tableName,
            TableMetadata tableMetadata
    ) throws SQLException {

        try (
            ResultSet rs = metaData.getPrimaryKeys(
                    null,
                    schemaName,
                    tableName
            )
        ) {

            while (rs.next()) {

                String columnName =
                        rs.getString("COLUMN_NAME");

                tableMetadata.addPrimaryKey(columnName);
            }
        }
    }
}
```

This class now dynamically inspects a table without hardcoding its column definitions.

## 7.10.16 — Run the Introspection Service
Let's test it against PostgreSQL.
### Main.java

```
package com.example;

import java.sql.Connection;
import java.sql.DriverManager;

import com.example.metadata.ColumnMetadata;
import com.example.metadata.DatabaseIntrospectionService;
import com.example.metadata.TableMetadata;

public class Main {

    public static void main(String[] args) {

        String url =
                "jdbc:postgresql://localhost:5432/employee_db";

        String username = "postgres";
        String password = "your_password";

        try (Connection connection =
                DriverManager.getConnection(
                        url,
                        username,
                        password
                )) {

            DatabaseIntrospectionService service =
                    new DatabaseIntrospectionService(connection);

            TableMetadata table =
                    service.inspectTable(
                            "public",
                            "employees"
                    );

            System.out.println(
                    "Table: " +
                    table.getSchemaName() +
                    "." +
                    table.getTableName()
            );

            System.out.println("\nColumns:");

            for (ColumnMetadata column : table.getColumns()) {

                System.out.println(
                        "Name: " + column.getColumnName() +
                        ", Type: " + column.getTypeName() +
                        ", JDBC Type: " + column.getJdbcType() +
                        ", Size: " + column.getColumnSize() +
                        ", Scale: " + column.getDecimalDigits() +
                        ", Nullable: " + column.isNullable() +
                        ", Position: " + column.getOrdinalPosition()
                );
            }

            System.out.println("\nPrimary Keys:");

            for (String primaryKey :
                    table.getPrimaryKeys()) {

                System.out.println(primaryKey);
            }

        } catch (Exception e) {

            e.printStackTrace();
        }
    }
}
```

Example output
```
Table: public.employees

Columns:
Name: id, Type: int4, JDBC Type: 4, Size: 32, Scale: 0, Nullable: false, Position: 1
Name: name, Type: varchar, JDBC Type: 12, Size: 100, Scale: 0, Nullable: true, Position: 2
Name: email, Type: varchar, JDBC Type: 12, Size: 255, Scale: 0, Nullable: true, Position: 3
Name: salary, Type: numeric, JDBC Type: 2, Size: 10, Scale: 2, Nullable: true, Position: 4
Name: joining_date, Type: date, JDBC Type: 91, Size: 13, Scale: 0, Nullable: true, Position: 5

Primary Keys:
id
```

Output is illustrative; PostgreSQL JDBC metadata values may vary.


## 7.10.17 — Discover All Tables and Inspect Them Dynamically
Until now, we inspected a table whose name we already knew.

But what if you want to inspect the entire database?

You can combine:

```
metaData.getTables()
```

with:
inspectTable()


### Workflow
```
Database
   │
   ▼
Discover all tables
   │
   ▼
Loop through tables
   │
   ├── employees
   │      └── Inspect columns + primary keys
   │
   ├── departments
   │      └── Inspect columns + primary keys
   │
   ├── projects
   │      └── Inspect columns + primary keys
   │
   └── audit_logs
          └── Inspect columns + primary keys
```

Example:

```
DatabaseMetaData metaData =
        connection.getMetaData();

try (
    ResultSet rs = metaData.getTables(
            null,
            "public",
            "%",
            new String[]{"TABLE"}
    )
) {

    while (rs.next()) {

        String tableName =
                rs.getString("TABLE_NAME");

        TableMetadata table =
                service.inspectTable(
                        "public",
                        tableName
                );

        System.out.println(
                "Inspected: " + tableName
        );

        System.out.println(
                "Column count: " +
                table.getColumns().size()
        );

        System.out.println(
                "Primary keys: " +
                table.getPrimaryKeys()
        );
    }
}
```

Now your application can inspect an unknown database dynamically.

This is the foundation of:
- Schema discovery
- Database migration tools
- Schema comparison
- Dynamic SQL generation
- JSON-to-DDL applications
- Data integration systems

## 7.10.18 — Understanding JDBC Types
One of the most useful things metadata gives you is the JDBC type code.

For example:
```
int jdbcType = rs.getInt("DATA_TYPE");
```

The value corresponds to constants in:
java.sql.Types


For example:

```
Types.INTEGER
Types.VARCHAR
Types.NUMERIC
Types.DATE
Types.TIMESTAMP
Types.BOOLEAN
Types.ARRAY
Types.OTHER


Instead of writing:
if (jdbcType == 4) {
    // INTEGER
}


You should write:
if (jdbcType == Types.INTEGER) {
    // INTEGER
}
```

Because this is much more readable.
Example JDBC type detection

```
import java.sql.Types;

public static String getTypeCategory(int jdbcType) {

    return switch (jdbcType) {

        case Types.INTEGER,
             Types.SMALLINT,
             Types.TINYINT ->
                "INTEGER";

        case Types.BIGINT ->
                "BIGINT";

        case Types.NUMERIC,
             Types.DECIMAL ->
                "DECIMAL";

        case Types.VARCHAR,
             Types.CHAR,
             Types.LONGVARCHAR ->
                "STRING";

        case Types.DATE ->
                "DATE";

        case Types.TIME,
             Types.TIME_WITH_TIMEZONE ->
                "TIME";

        case Types.TIMESTAMP,
             Types.TIMESTAMP_WITH_TIMEZONE ->
                "TIMESTAMP";

        case Types.BOOLEAN,
             Types.BIT ->
                "BOOLEAN";

        default ->
                "OTHER";
    };
}
```

This is useful when you want to classify database columns into groups.

For example:

```
INTEGER
STRING
DECIMAL
DATE
TIME
TIMESTAMP
BOOLEAN
OTHER
```

Later, you can use these categories to determine:
- Which columns should be excluded from UUID generation
- Which columns should be treated as numeric
- Which columns can be used for comparisons
- Which columns need date/time handling
- Which columns can be safely converted to strings

## 7.10.19 — Database Metadata vs PostgreSQL System Catalogs
JDBC metadata provides a standardized abstraction.

However, PostgreSQL also has its own system catalogs.

Examples:
```
information_schema.tables
information_schema.columns
information_schema.table_constraints
information_schema.key_column_usage
```

And PostgreSQL-specific catalog tables:
```
pg_catalog.pg_class
pg_catalog.pg_attribute
pg_catalog.pg_constraint
pg_catalog.pg_index
pg_catalog.pg_namespace
```

#### JDBC Metadata

Database portable
- Standard JDBC API
- Cross-database abstraction
- Easier to maintain
- Less database-specific detail


#### PostgreSQL Catalogs

PostgreSQL-specific
- Detailed internal metadata
- Advanced constraints
- Index definitions
- Expression indexes
- PostgreSQL-specific type details



### When to use JDBC Metadata?
Use JDBC metadata when:
- You want database portability.
- You need general schema discovery.
- You are creating a generic database tool.
- Standard metadata is sufficient.

### When to use PostgreSQL system catalogs?
Use system catalogs when:
- You need detailed PostgreSQL-specific information.
- You need exact DDL definitions.
- You need advanced index information.
- You need information not exposed by JDBC metadata.
A professional database tool often uses both.

## 7.10.20 — Important Introspection Limitations
JDBC metadata is powerful, but it isn't perfect for every database-specific requirement.

Keep these limitations in mind:

1. Metadata can differ between database drivers
PostgreSQL, MySQL, Oracle, and SQL Server may report some fields differently.

2. Column size is not always what you expect
For example, COLUMN_SIZE may represent:
- Character length
- Numeric precision
- A driver-specific size representation
Do not assume that every database returns identical semantics.

3. Type names can be database-specific
For example:
PostgreSQL: int4, varchar, jsonb, uuid

While other databases may report different type names.

4. Metadata visibility depends on permissions
A database user may not have visibility into every object.

5. Generated columns and identity columns
Driver support and metadata behavior may differ. If you need accurate PostgreSQL-specific generation details, you may need system catalog queries.

6. Primary keys may be composite
Never assume:
String primaryKey;


A table may have:
List<String> primaryKeys;


7. Foreign keys may reference multiple columns
You should preserve:
- Constraint name
- FK column
- Referenced table
- Referenced column
- Sequence position

## 7.10.21 — How Database Introspection Is Used in Real Applications
This is where the topic becomes more interesting than simply calling JDBC methods.

Imagine a tool where the user connects to a database, and your Java application automatically displays its structure.

The user doesn't tell you the table names or columns.

Your application discovers everything.

Example architecture

```
          Database Connection
                  │
                  ▼
       DatabaseIntrospectionService
                  │
        ┌─────────┼──────────┐
        ▼         ▼          ▼
     Tables    Columns   Constraints
        │         │          │
        └─────────┼──────────┘
                  ▼
           Metadata Models
                  │
          ┌───────┼─────────┐
          ▼       ▼         ▼
      JSON API  Schema    SQL
                 Viewer  Generator
```

You can use this architecture to build a tool that supports:
- Database schema viewer
- Schema export to JSON
- ER diagram generation
- Dynamic table information
- SQL generation
- Data migration
- Data replication configuration

## 7.10.22 — Practical Project: Database Schema Explorer
Mini-project assignment

Build a pure Java application that can connect to PostgreSQL and inspect a database dynamically.
Requirements
Part 1 — Database Information
Display:
- Database name
- Database version
- JDBC driver
- Driver version
- Connection URL
Part 2 — Schema Discovery
Discover all schemas.
metaData.getSchemas();


Part 3 — Table Discovery
Discover all tables in a selected schema.
metaData.getTables(
    null,
    "public",
    "%",
    new String[]{"TABLE"}
);


Part 4 — Column Discovery
For every table, discover:
- Column name
- Type name
- JDBC type
- Precision/size
- Scale
- Nullable
- Default value
- Ordinal position
Part 5 — Primary Keys
Discover all primary key columns.
Part 6 — Foreign Keys
Discover relationships between tables.
Part 7 — Indexes
Discover indexes and indexed columns.
Part 8 — Metadata JSON Output
Convert the metadata into JSON.

Example output:
{
  "schema": "public",
  "table": "employees",
  "columns": [
    {
      "name": "id",
      "type": "int4",
      "jdbcType": 4,
      "nullable": false,
      "position": 1,
      "primaryKey": true
    },
    {
      "name": "name",
      "type": "varchar",
      "jdbcType": 12,
      "nullable": true,
      "position": 2,
      "primaryKey": false
    }
  ]
}



## 7.10.23 — Questions

### Q1. What is database introspection?
Database introspection is the process of programmatically examining database structure, including schemas, tables, columns, data types, constraints, indexes, and relationships.
### Q2. What is DatabaseMetaData?
DatabaseMetaData is a JDBC interface that provides information about a database, its structure, and its JDBC driver's capabilities.
### Q3. How do you discover tables dynamically?
DatabaseMetaData metaData = connection.getMetaData();
```
ResultSet rs = metaData.getTables(
        null,
        "public",
        "%",
        new String[]{"TABLE"}
);
```

### Q4. How do you discover columns?
ResultSet rs = metaData.getColumns(
        null,
        "public",
        "employees",
        "%"
);


### Q5. What is ResultSetMetaData?
It provides information about the columns of a query's ResultSet, such as names, types, precision, scale, and column count.
### Q6. Difference between DatabaseMetaData and ResultSetMetaData?
- DatabaseMetaData → database structure
- ResultSetMetaData → structure of a query result
### Q7. How do you discover primary keys?
metaData.getPrimaryKeys(
        null,
        "public",
        "employees"
);


### Q8. How do you discover foreign keys?
metaData.getImportedKeys(
        null,
        "public",
        "employees"
);


### Q9. Why is database introspection useful?
It allows applications to work with unknown or changing database structures without hardcoding table and column definitions.
### Q10. What is the difference between JDBC metadata and PostgreSQL system catalogs?
JDBC metadata is a standardized interface designed for portability. PostgreSQL system catalogs expose deeper PostgreSQL-specific information.
