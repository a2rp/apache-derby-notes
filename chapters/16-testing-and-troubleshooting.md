# 16. Testing and troubleshooting

[Back to notes index](../README.md)

| [Previous: Authentication, authorization, and security](15-authentication-authorization-and-security.md) | [Notes index](../README.md) | [Next: Performance and operating considerations](17-performance-and-operations.md) |
| --- | --- | --- |

## Test against an isolated database

Database tests should not depend on your personal working database or a production database. Use a separate database with a known schema and predictable rows. Give each run a clean database location, then remove it only after Derby has shut it down. This makes failures repeatable and prevents a test from changing real data.

Create a small test database with `ij`:

```sql
CONNECT 'jdbc:derby:library_test;create=true';

CREATE TABLE APP.TEST_BOOKS (
    BOOK_ID INTEGER NOT NULL PRIMARY KEY,
    TITLE VARCHAR(120) NOT NULL
);

INSERT INTO APP.TEST_BOOKS (BOOK_ID, TITLE)
VALUES (1, 'Test Book');

SELECT BOOK_ID, TITLE
FROM APP.TEST_BOOKS;
```

An integration test can open this test database through JDBC and verify results using the same driver and SQL that the application uses. Keep setup data small and explicit. If one test needs a different starting state, create that state in its own setup code or roll back its transaction at the end.

Use a prepared statement for values and assert the result:

```java
String sql = "SELECT TITLE FROM APP.TEST_BOOKS WHERE BOOK_ID = ?";

try (PreparedStatement statement = connection.prepareStatement(sql)) {
    statement.setInt(1, 1);

    try (ResultSet results = statement.executeQuery()) {
        if (!results.next()) {
            throw new AssertionError("Expected one test book");
        }

        String title = results.getString("TITLE");
        if (!"Test Book".equals(title)) {
            throw new AssertionError("Unexpected title: " + title);
        }
        if (results.next()) {
            throw new AssertionError("Expected exactly one matching row");
        }
    }
}
```

This check does not rely on Java's optional `assert` keyword, which is disabled unless the JVM starts with `-ea`. Close statements, result sets, and connections with try-with-resources so a failed check does not leave resources open.

## Test constraints and transaction behavior

Test both accepted and rejected data. A primary key should accept a unique key and reject a duplicate. A `NOT NULL` column should reject a missing value. A foreign key should reject a reference to a row that does not exist. Run these checks only in the disposable test database.

For example, after inserting `BOOK_ID = 1` in `APP.TEST_BOOKS`, this insert should fail with an integrity constraint error because that key already exists:

```sql
INSERT INTO APP.TEST_BOOKS (BOOK_ID, TITLE)
VALUES (1, 'Duplicate key');
```

If the test must preserve existing rows, run the test work in a transaction and roll it back:

```java
connection.setAutoCommit(false);
try {
    // Perform test inserts and updates here.
    connection.rollback();
} catch (SQLException exception) {
    connection.rollback();
    throw exception;
} finally {
    connection.setAutoCommit(true);
}
```

Do not use rollback as a substitute for an isolated test database. DDL and stored procedures can have transaction behavior that differs from ordinary inserts and updates, and a failed test must never leave shared data uncertain.

## Read Derby exceptions in JDBC

When JDBC reports an error, inspect the SQLState and message before changing code. Derby can return more than one `SQLException` for a failure. The first exception is often the most severe, while a later exception may contain a more specific cause. Walk the exception chain:

```java
try {
    // Run a statement that uses the database.
} catch (SQLException exception) {
    for (SQLException current = exception;
         current != null;
         current = current.getNextException()) {
        System.err.printf(
                "SQLState=%s, code=%d, message=%s%n",
                current.getSQLState(),
                current.getErrorCode(),
                current.getMessage());
    }
}
```

Use SQLState to classify a failure, such as a connection problem, invalid SQL, a constraint violation, or a transaction rollback. Read the full message and chain as well. Derby's numeric error code represents severity and is not a unique error identifier, so application logic should not depend on it.

If Derby returns a `SQLWarning`, inspect it too. A warning does not necessarily mean the statement failed. For example, Derby can return a warning when `create=true` connects to a database that already exists.

## Check a table after a suspected storage issue

`SYSCS_UTIL.SYSCS_CHECK_TABLE` checks a table's internal consistency and verifies that its indexes agree with the base table. Run it when investigating a suspected storage problem or validating a backup copy. It can take a long time on a large table, so it is not a routine check to run after every query.

```sql
VALUES SYSCS_UTIL.SYSCS_CHECK_TABLE('LIBRARY', 'BOOKS');
```

A non-zero result means the table is consistent. An inconsistency raises an exception. After a backup, keep the previous known-good copy until the new copy has been checked and a restore test succeeds.

## Inspect a query plan

When a query is slower than expected, inspect Derby's runtime statistics in the same connection. Enable collection, run the query, read the plan, then turn collection off:

```sql
CALL SYSCS_UTIL.SYSCS_SET_RUNTIMESTATISTICS(1);

SELECT BOOK_ID, TITLE
FROM LIBRARY.BOOKS
WHERE PUBLISHED_YEAR >= 2000;

VALUES SYSCS_UTIL.SYSCS_GET_RUNTIMESTATISTICS();

CALL SYSCS_UTIL.SYSCS_SET_RUNTIMESTATISTICS(0);
```

The result shows the execution plan and statistics such as the rows read and returned. In `ij`, increase the display width before reading the result so long lines are not cut off:

```text
MaximumDisplayWidth 5000
```

Statistics are connection-specific. Enable timing only when you need execution timing, because timing is off by default:

```sql
CALL SYSCS_UTIL.SYSCS_SET_RUNTIMESTATISTICS(1);
CALL SYSCS_UTIL.SYSCS_SET_STATISTICS_TIMING(1);
```

Compare plans after changing a query or index. Do not assume that adding an index always improves a query; inspect the actual predicates, row counts, and plan.

## Use the Derby tools to narrow down a problem

The Derby distribution includes tools for common checks:

| Tool | Use |
| --- | --- |
| `ij` | Open a connection, run SQL, and execute a saved SQL script. |
| `sysinfo` | Check the Java environment and Derby libraries visible to the process. |
| `dblook` | Generate DDL from a database to review its schema. |
| `NetworkServerControl` | Start, stop, or inspect the Network Server. |

Run `sysinfo` when a driver cannot load or different Derby versions may be on the classpath:

```text
java -jar "${env:DERBY_HOME}/lib/derbyrun.jar" sysinfo
```

Use `dblook` to inspect the DDL for a local database:

```text
java -jar "${env:DERBY_HOME}/lib/derbyrun.jar" dblook -d "jdbc:derby:librarydb"
```

`dblook` prints generated DDL to the console by default. Check the Derby Tools and Utilities Guide for its schema filters and output options. Avoid putting production passwords directly in shell history when connecting to an authenticated database.

## Troubleshoot in a consistent order

1. Read the complete exception chain, SQLState, and message.
2. Confirm the JDBC URL, database path, driver, and Java version.
3. Use `sysinfo` to check the libraries loaded by the failing process.
4. Confirm the database is booted and the expected schema and table exist in `ij`.
5. Check the current user and its table or role privileges if the error is an authorization failure.
6. Reproduce the error with the smallest SQL statement and the same connection mode.
7. Inspect runtime statistics only if the issue is query behavior or speed.
8. Back up the database before attempting a repair or changing stored data.

For Network Server failures, verify that the server is running, the client uses the right host and port, the database name is correct, and firewall rules allow the connection. For lock waits or deadlocks, keep transactions short, access tables in a consistent order, and handle transaction rollback by starting a new transaction when appropriate.

## Common mistakes

- Running a test against the only copy of important data.
- Relying on `assert` while forgetting that it is disabled by default.
- Leaving a result set, statement, or connection open after a failed test.
- Looking only at the first SQL exception and missing the next exception in the chain.
- Treating Derby's error code as a unique identifier.
- Running a full table consistency check on a large database after every change.
- Collecting runtime statistics in one connection and reading them from another.
- Using `dblook` or `ij` from an old Derby installation on the classpath.
- Trying to repair database files before creating a safe backup.

## Practice

1. Create the `library_test` database and `APP.TEST_BOOKS` table with `ij`.
2. Run the JDBC result check and verify that an incorrect expected title makes it fail.
3. Try the duplicate-key insert and record the SQLState and full exception message.
4. Run `sysinfo` and confirm which Derby version and jar files are in use.
5. Generate the test database DDL with `dblook` and compare it with the statements you wrote.
6. Enable runtime statistics, run a filtered books query, and identify the scan or index access in the plan.
7. On a backup copy, run `SYSCS_CHECK_TABLE` for `LIBRARY.BOOKS` and record the returned value.

## Further reading

- [Working with Derby SQLExceptions](https://db.apache.org/derby/docs/10.17/devguide/cdevconcepts844813.html)
- [SQLException reference](https://db.apache.org/derby/docs/10.17/ref/rrefjdbc16643.html)
- [Derby tools and startup utilities](https://db.apache.org/derby/docs/10.17/getstart/cgsusingtoolsutils.html)
- [Running dblook](https://db.apache.org/derby/docs/10.17/getstart/tgsrunningdblook.html)
- [SYSCS_DIAG.ERROR_MESSAGES](https://db.apache.org/derby/docs/10.17/ref/rrefsyscsdiagerrormessages.html)
- [SYSCS_UTIL.SYSCS_CHECK_TABLE](https://db.apache.org/derby/docs/10.17/ref/rrefsyscschecktablefunc.html)
- [Runtime statistics in the Reference Manual](https://db.apache.org/derby/docs/10.17/ref/refderby.pdf)
- [Derby Tuning Guide](https://db.apache.org/derby/docs/10.17/tuning/)

| [Previous: Authentication, authorization, and security](15-authentication-authorization-and-security.md) | [Notes index](../README.md) | [Next: Performance and operating considerations](17-performance-and-operations.md) |
| --- | --- | --- |
