# Database Connection

### What is JDBC?

JDBC (Java Database Connectivity) is Java's standard API for interacting with relational databases. It provides interfaces such as Connection, Statement, PreparedStatement, and ResultSet. A database-specific JDBC driver implements these interfaces and translates Java database operations into the database's native protocol.

### Why do we prefer PreparedStatement over Statement?

PreparedStatement supports parameterized queries using placeholders (?). It separates SQL from user input, which helps prevent SQL injection attacks. It can also improve performance because the database can reuse the compiled execution plan when the same query is executed multiple times with different values.