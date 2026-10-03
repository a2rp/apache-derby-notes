# 99. Complete questions and answers

[Back to notes index](../README.md)

| [Previous: All code samples](98-all-code-samples.md) | [Notes index](../README.md) | [Next: Notes index](../README.md) |
| --- | --- | --- |

Use these questions to review the core ideas from the notes. Answers focus on the reason behind each concept. For full SQL and Java examples, open the linked chapter or the [all code samples appendix](98-all-code-samples.md).

## 1. Relational databases and Derby

### 1. What does a relational database store?

It stores related data in tables. A table has columns that describe each value and rows that hold individual records.

### 2. Why does a table need a primary key?

A primary key identifies each row uniquely and cannot be `NULL`. Other tables can refer to that stable identifier instead of copying all of the row's values.

### 3. What does a foreign key protect?

A foreign key requires a value in one table to match a key in a referenced table. It prevents a row from pointing to a record that does not exist.

### 4. What is a Derby database?

It is a set of database files managed by the Derby engine. Applications connect to it through Derby's JDBC driver.

### 5. How do embedded and Network Server modes differ?

In embedded mode, the application and Derby engine run in the same Java process. In Network Server mode, a separate Derby server process accepts JDBC connections from clients.

### 6. Does Derby require a separate database server for embedded use?

No. An embedded application loads the Derby engine directly. The Network Server is only needed when separate client processes must connect over a network.

## 2. Java compatibility, setup, and tools

### 7. Which Derby version do these notes use?

The examples use Derby 10.17.1.0. That release is listed for Java 21 and higher, so confirm the Java requirement before selecting jars for an older runtime.

### 8. What is `DERBY_HOME`?

It is a convenient environment variable pointing to the Derby installation directory. The `lib` folder contains jars, and the `bin` folder contains command scripts in the binary distribution.

### 9. Which jar is needed for embedded JDBC?

The embedded engine and driver are in `derby.jar`. Tools such as `ij` and `dblook` also need the Derby tools classes, which are included in the distribution's tools jars or launch setup.

### 10. What does `ij` do?

`ij` is a command-line JDBC tool for connecting to Derby and running SQL statements or scripts. It is useful for checking database setup without writing an application first.

### 11. What does `sysinfo` tell me?

It reports details about the Java environment and Derby libraries that the process can see. Use it to investigate a missing driver or multiple Derby versions on the classpath.

### 12. What is the current maintenance status of Derby?

The Apache project moved Derby to retired, read-only status on October 10, 2025. Development and bug fixes have ended, and no further releases are planned.

## 3. Using `ij` and creating a database

### 13. How do I create a database from `ij`?

Connect with a JDBC URL such as `jdbc:derby:librarydb;create=true`. Derby creates the database if it does not already exist.

### 14. Is `CREATE DATABASE` a Derby SQL statement?

No. Create or access a Derby database through connection URL attributes such as `create=true`.

### 15. Where does Derby put a relative database path?

It uses the configured Derby system home or the process working directory, depending on configuration. Use a deliberate system home or an absolute path so the database location is clear.

### 16. How do I run a saved SQL file?

Start `ij` with the script file path or use `ij`'s run command. Keep statements terminated correctly and include the `CONNECT` command if the script needs a particular database.

### 17. Why might a shutdown connection report an exception?

Derby reports that the database has shut down through the JDBC connection attempt. For the documented shutdown URL, that exception is the expected confirmation that shutdown completed.

### 18. Can I delete a database while Derby is using it?

No. Shut the database down first. Removing files while the engine has the database open can damage the database.

## 4. Schemas, tables, and data types

### 19. What is a schema?

A schema is a namespace for database objects such as tables, views, indexes, and routines. Qualifying a table as `LIBRARY.BOOKS` makes its schema explicit.

### 20. What is Derby's common default schema?

For a new user it is commonly `APP`. The current schema affects how Derby resolves unqualified object names.

### 21. What is the difference between `NULL` and an empty string?

`NULL` means a value is absent or unknown. An empty string is a present text value with zero characters.

### 22. When should I use `DECIMAL` instead of `DOUBLE`?

Use `DECIMAL` for exact decimal values such as prices. `DOUBLE` is approximate and can produce small rounding differences.

### 23. When is an identity column useful?

Use it when Derby should generate a numeric key for each inserted row. Generated values can have gaps, so do not treat them as a gap-free count.

### 24. Why use `DATE` or `TIMESTAMP` instead of text for dates?

Date and timestamp types support date comparisons, sorting, and date operations. Text dates can sort incorrectly when their formats differ.

## 5. Constraints and indexes

### 25. What is a `NOT NULL` constraint for?

It requires every row to provide a value for that column. Use it when a record is incomplete or invalid without that value.

### 26. What is the difference between `UNIQUE` and a primary key?

Both prevent duplicate key values. A primary key also identifies the table's main row key and cannot contain `NULL` values.

### 27. What does a `CHECK` constraint do?

It limits values to a condition that must be true for each row. For example, a year can be required to be positive or within an accepted range.

### 28. Why might Derby create an index automatically?

Primary key and unique constraints use an index to enforce uniqueness. The referenced side of a foreign key must have a primary key or unique key; an index on the referencing columns can also help queries and deletes that look up related rows.

### 29. Should I index every column used in a query?

No. Each index uses storage and adds work to inserts, updates, and deletes. Add indexes for measured query patterns and compare the execution plan.

### 30. What is a composite index?

It is one index containing multiple columns in a defined order. The order affects which filters and sorts can use it, so choose it based on actual query patterns.

## 6. Inserting, updating, and deleting data

### 31. How can I insert only selected columns?

Name the target columns in the `INSERT` statement and provide values in the same order. Other columns need defaults or must allow `NULL`.

### 32. What happens if an `UPDATE` has no `WHERE` clause?

It updates every row in the table. Check the predicate and target rows before running an update against important data.

### 33. What happens if a `DELETE` has no `WHERE` clause?

It deletes every row in the table. It keeps the table definition, unlike dropping the table.

### 34. How do I update one row safely?

Use a predicate that identifies the intended row, preferably through a primary key. In JDBC, bind the key with a prepared statement and check the affected-row count.

### 35. What should an application do if an insert violates a constraint?

Catch the `SQLException`, inspect its SQLState and message, and decide whether to show a validation error or roll back the transaction. Do not silently ignore the failure.

### 36. What does `COMMIT` do?

It makes the current transaction's changes permanent and releases the transaction's locks. With JDBC auto-commit enabled, Derby commits each completed statement automatically.

## 7. Selecting and filtering data

### 37. Why should a query list its columns instead of using `SELECT *`?

It returns only the fields the caller needs and makes the result shape clear. It also avoids unexpected extra data when the table changes.

### 38. How do I test whether a column has no value?

Use `IS NULL` or `IS NOT NULL`. Comparing with `= NULL` does not test for missing values correctly.

### 39. Does a query have a guaranteed row order without `ORDER BY`?

No. Specify `ORDER BY` whenever the application depends on a particular order.

### 40. What do `%` and `_` mean in a `LIKE` pattern?

`%` matches any number of characters, including none. `_` matches exactly one character.

### 41. How do I return only the first few rows?

Use Derby's `FETCH FIRST n ROWS ONLY` clause. Pair it with `ORDER BY` when you need a predictable set of rows.

### 42. Why should Java bind values with `PreparedStatement`?

It keeps values separate from SQL syntax, handles type conversion, and helps prevent input from changing the query structure. It does not bind table or column names.

## 8. Joins, subqueries, and views

### 43. What rows does an inner join return?

It returns rows with matching join values in both inputs. Rows without a match are omitted.

### 44. When is a left join useful?

It keeps every row from the left table and adds matching values from the right table when they exist. The right-side columns are `NULL` when there is no match.

### 45. Why should a join condition compare related keys?

It expresses the relationship between the tables. Missing or incorrect join conditions can create many unintended row combinations.

### 46. What is a correlated subquery?

It refers to a value from the outer query. It is evaluated in relation to each candidate outer row, so check the plan and result carefully for large inputs.

### 47. When can `EXISTS` be clearer than a join?

Use `EXISTS` when the question is whether at least one related row exists and no columns from that related row are needed in the result.

### 48. Is a Derby view a stored copy of its query results?

No. A view stores a query definition and presents its results when queried. Derby views are not updatable, and a view defined with `SELECT *` does not automatically gain columns added to its base table.

## 9. Transactions and concurrency

### 49. What is a transaction?

It is a group of database operations treated as one unit. A transaction either commits its changes or rolls them back.

### 50. What does JDBC auto-commit mean?

Each completed statement is committed automatically. Turn auto-commit off when several statements must succeed or fail together.

### 51. When should an application call `rollback()`?

Call it when a transaction cannot safely complete. It undoes the uncommitted work in that transaction.

### 52. What is isolation in a database?

Isolation controls which changes one transaction can observe while other transactions are running. More restrictive isolation can reduce anomalies but can increase waiting.

### 53. What is a deadlock?

It is a cycle where transactions each wait for a lock held by another transaction in the cycle. Derby aborts a transaction so the remaining work can proceed.

### 54. How can an application reduce lock contention?

Keep transactions short, access shared tables in a consistent order, and avoid waiting for network or user activity while holding locks.

## 10. JDBC connections and prepared statements

### 55. What does JDBC provide?

JDBC is Java's API for opening database connections, executing SQL, and reading results. Derby supplies JDBC drivers for embedded and Network Server connections.

### 56. Why use `try` with resources for JDBC objects?

It closes a `Connection`, `Statement`, and `ResultSet` when the code finishes or fails. Leaving them open can hold locks and database resources.

### 57. Which number does the first prepared-statement parameter use?

Parameters are numbered from `1`. Bind each value using the setter that matches its type.

### 58. Should user input be concatenated into SQL?

No. Use placeholders and bind values with a prepared statement. Build dynamic identifiers only from a controlled allowlist because parameters represent values, not SQL identifiers.

### 59. How do I know how many rows an update changed?

Read the integer returned by `executeUpdate()`. Use it to confirm that the expected number of rows matched the statement.

### 60. How do I read a query result?

Call `executeQuery()` and advance its `ResultSet` with `next()`. Read each current row by column label or index before advancing again.

## 11. Embedded mode and database lifecycle

### 61. Where does the embedded Derby engine run?

Inside the application's Java process. The application and database engine share a JVM and load the embedded driver.

### 62. Can two unrelated processes open the same embedded database at once?

An embedded database is normally owned by the process that booted it. Use Network Server mode when separate client processes need concurrent access to one database.

### 63. What does `derby.system.home` control?

It sets the base directory Derby uses for relative database names and system files. Configure it before the engine starts so databases are created in a predictable place.

### 64. How do I shut down one embedded database?

Use the database connection URL with `shutdown=true` and the database name. Derby closes that database and reports the shutdown through the connection attempt.

### 65. How do I shut down the full Derby engine?

Use the system shutdown URL without a database name, such as `jdbc:derby:;shutdown=true`. The exact shutdown procedure depends on embedded or Network Server mode.

### 66. Can I move a database directory while Derby is running?

No. Shut the database down cleanly first, or use Derby's online backup procedure to create a consistent copy.

## 12. Network Server and client connections

### 67. What is the role of the Network Server?

It hosts the Derby engine and accepts remote JDBC client connections. It lets multiple client processes connect to the same database through the server.

### 68. What is the role of the Network Client driver?

It sends JDBC requests from the application to a Derby Network Server. It does not host the database engine itself.

### 69. What does a network JDBC URL look like?

It commonly looks like `jdbc:derby://host:1527/databaseName`. Replace the host and database name with the actual server and database.

### 70. Where does a server-side import or export file path refer to?

It refers to the server machine's file system, because the Derby engine running on the server reads or writes the file.

### 71. Should the Network Server listen on every network interface by default?

No. Keep it on localhost unless remote connections are needed. If remote access is enabled, restrict it with firewall rules and protect the connection with SSL/TLS.

### 72. What should I check when a client cannot connect?

Check that the server is running, the host and port are correct, the database name exists, network rules allow the connection, and the client has the matching Derby network driver.

## 13. Metadata and schema changes

### 73. What can `DatabaseMetaData` tell an application?

It can report tables, columns, keys, driver details, and other database structure information through JDBC. It is useful when the application must inspect a database at runtime.

### 74. Why does Derby metadata code often pass `null` for catalog?

Derby does not use catalogs in the same way as some databases. Pass `null` for the catalog argument and provide the schema and table patterns.

### 75. Why do metadata searches often use uppercase names?

Unquoted Derby identifiers are normalized to uppercase. Metadata patterns should match the stored form, such as `LIBRARY` and `BOOKS`.

### 76. Should application code write to `SYS` catalog tables?

No. Treat system catalogs as read-only sources for inspection. Use supported SQL statements and JDBC metadata methods to change or inspect schema objects.

### 77. Why should a migration be tested before applying it?

A migration can fail because existing rows violate a new constraint or because a dependent view or query expects an old column. Test it against a copy and back up important data first.

### 78. Does a view using `SELECT *` gain columns when its table changes?

No. Its result columns are fixed when the view is created. Recreate the view if it should include a newly added base-table column.

## 14. Import, export, backup, and recovery

### 79. When should I export rows instead of backing up the database?

Export rows when moving or inspecting table data in a delimited file. Use a database backup when you need a consistent copy of the complete Derby database.

### 80. Can `SYSCS_EXPORT_TABLE` overwrite an existing file?

No. The export destination must be a new file. Choose a new path or move the previous file before exporting again.

### 81. What must be true before importing a file?

The target table must already exist, and the input columns, order, types, and delimiters must match the import call. Table constraints still apply to imported rows.

### 82. What is the difference between import insert and replace modes?

Insert mode appends imported rows to existing table data. Replace mode removes all existing rows before loading the file, while keeping the table definition and indexes.

### 83. Why should I complete a transaction before an import or export procedure?

Derby commits after a successful import or export procedure and rolls back after a failure. Finish unrelated work first so the procedure does not commit it unexpectedly.

### 84. What does an online backup contain?

It copies the full database directory to a backup location while Derby is running. It is different from a delimited file export of one table.

### 85. Is restoring a full backup safe over an existing database?

It replaces a same-named database in the configured Derby home. Confirm the target and rehearse on an isolated copy before restoring important data.

### 86. What does roll-forward recovery require?

It requires a full backup plus the archived and active log files needed to replay transactions since the backup. It restores the whole database, not a single table.

## 15. Authentication, authorization, and security

### 87. What is the difference between authentication and authorization?

Authentication checks who is connecting. Authorization checks which database operations that identity may perform.

### 88. What is NATIVE authentication?

It stores user names and encrypted passwords in a Derby database. Derby can use that database's credentials to authenticate users.

### 89. Which user must be created first when enabling NATIVE authentication?

The database owner must be the first user whose credentials are stored. Derby enables NATIVE authentication after the database is shut down and booted again.

### 90. Does NATIVE authentication also enable SQL authorization?

Yes. Derby enables fine-grained SQL authorization with NATIVE authentication, so the database owner can use roles and `GRANT` statements to assign privileges.

### 91. Why grant privileges through roles?

Roles group permissions by job, such as read-only access or the ability to add and update books. This is easier to review and narrower than giving each user broad access.

### 92. What is the risk of granting a privilege to `PUBLIC`?

The privilege applies to every current and future user. Grant it only when that broad access is intended.

### 93. Does a password protect Network Server traffic from being read?

No. Authentication verifies identity, but transport encryption requires SSL/TLS. Restrict remote access with a firewall as well.

### 94. Where should an application store database credentials?

Use a protected configuration source supplied by the runtime environment. Do not commit passwords, place them in public URLs, or write them to logs.

## 16. Testing and troubleshooting

### 95. Why should database tests use an isolated database?

An isolated database gives each test predictable data and prevents tests from changing a real working database. It also makes failures easier to repeat.

### 96. What information should I inspect in a Derby `SQLException`?

Inspect the SQLState, message, error code, and chained exceptions. The next exception can contain details that are not in the first one.

### 97. Is Derby's integer error code a unique error identifier?

No. It represents error severity and is not unique to each error. Use the SQLState and message to understand the failure.

### 98. What does `SYSCS_UTIL.SYSCS_CHECK_TABLE` return for a consistent table?

It returns a non-zero `SMALLINT`, normally `1`. If Derby finds an inconsistency, it throws an exception.

### 99. How do I inspect a query's execution plan in Derby?

Enable runtime statistics on the connection, run the query, and read `SYSCS_UTIL.SYSCS_GET_RUNTIMESTATISTICS()`. Turn statistics collection off after the inspection.

### 100. Which Derby tools help diagnose setup problems?

Use `ij` to connect and run SQL, `sysinfo` to inspect Java and Derby libraries, `dblook` to print schema DDL, and `NetworkServerControl` for server operations.

### 101. Should I run a table consistency check after every query?

No. It can take a long time on a large table. Use it when investigating a suspected storage issue or checking a backup copy.

## 17. Performance and operating considerations

### 102. What should I measure before adding an index?

Record the slow query, its input values, representative table size, and runtime plan. Then test whether an index improves reads enough to justify its storage and write costs.

### 103. Why does the order of columns matter in a composite index?

The index stores its columns in a defined order. A query can make the best use of its leading columns, so choose the order to match the filters and sorting used by real queries.

### 104. When should I refresh index statistics?

Refresh them after a substantial change in the distribution or number of distinct values in indexed columns. Stale estimates can lead the optimizer to choose a less useful plan.

### 105. What does `SYSCS_UTIL.SYSCS_COMPRESS_TABLE` do?

It rebuilds a table and its indexes to reclaim unused allocated space. It takes an exclusive table lock, so schedule it when applications can wait.

### 106. What does the sequential compression argument control?

A non-zero value makes Derby compress the table and indexes one at a time. This reduces temporary memory and disk space but takes longer.

### 107. Why should a transaction be short?

Locks remain held until commit or rollback. Short transactions release locks sooner and reduce the chance that other work must wait.

### 108. What should happen before a Derby database upgrade?

Back up and verify a copy, check the loaded Derby jars, and test the upgrade before changing the only working database. A full upgrade can prevent older Derby versions from opening that database.

### 109. Why is Derby's retired status relevant to new systems?

The project no longer publishes releases or bug fixes. That affects maintenance, security response, and support expectations for a new production system.

## Further reading

- [Apache Derby manuals](https://db.apache.org/derby/manuals/)
- [Apache Derby current status](https://db.apache.org/derby/derby_downloads)
- [Notes index](../README.md)

| [Previous: All code samples](98-all-code-samples.md) | [Notes index](../README.md) | [Next: Notes index](../README.md) |
| --- | --- | --- |
