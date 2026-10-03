# 17. Performance and operating considerations

[Back to notes index](../README.md)

| [Previous: Testing and troubleshooting](16-testing-and-troubleshooting.md) | [Notes index](../README.md) | [Next: All code samples](98-all-code-samples.md) |
| --- | --- | --- |

## Measure before changing a query

Start with the query that is slow and inspect its runtime statistics, as shown in the previous chapter. Record the query, input values, table size, and current plan before changing indexes or SQL. Test with representative data because a plan that works well for a small sample may not work well after the table grows.

Return only the columns the application uses and filter rows in SQL:

```sql
SELECT BOOK_ID, TITLE
FROM LIBRARY.BOOKS
WHERE PUBLISHED_YEAR >= 2000
ORDER BY PUBLISHED_YEAR, BOOK_ID;
```

This keeps the result smaller than `SELECT *` and lets Derby evaluate the filter and ordering. In application code, bind input values with a `PreparedStatement` instead of joining values into the SQL string:

```java
String sql = "SELECT BOOK_ID, TITLE "
        + "FROM LIBRARY.BOOKS "
        + "WHERE PUBLISHED_YEAR >= ? "
        + "ORDER BY PUBLISHED_YEAR, BOOK_ID";

try (PreparedStatement statement = connection.prepareStatement(sql)) {
    statement.setInt(1, firstYear);
    try (ResultSet results = statement.executeQuery()) {
        while (results.next()) {
            System.out.println(
                    results.getInt("BOOK_ID") + ": "
                            + results.getString("TITLE"));
        }
    }
}
```

## Add indexes for measured access patterns

An index can reduce the rows Derby needs to inspect when a query filters, joins, or sorts on indexed columns. Indexes also take disk space and make inserts, updates, and deletes do more work. Add an index to answer a measured query, then compare its plan and write cost.

For the filtered books query, test an index on the publication year:

```sql
CREATE INDEX LIBRARY.BOOKS_YEAR_IX
ON LIBRARY.BOOKS (PUBLISHED_YEAR);
```

For a query that filters by both author and year in another schema, a composite index may fit that access pattern:

```sql
CREATE INDEX LIBRARY.BOOKS_YEAR_TITLE_IX
ON LIBRARY.BOOKS (PUBLISHED_YEAR, TITLE);
```

Column order matters. Put the columns that your queries search together in an order that matches the predicates and sort requirements. A composite index is most useful when a query can use its leading columns. Check the actual plan instead of adding every queried column to one large index.

## Refresh index statistics when data changes shape

Derby uses cardinality statistics to estimate how many rows a predicate will match. Large changes in the number or distribution of distinct values can make an index's statistics less useful. After a substantial data change, the database or schema owner can refresh statistics for every index on a table:

```sql
CALL SYSCS_UTIL.SYSCS_UPDATE_STATISTICS(
    'LIBRARY', 'BOOKS', NULL
);
```

Run the query again and compare its runtime plan. Refreshing statistics does not guarantee a faster plan; it gives Derby updated information to choose a plan. Do this when data changes enough to make the old estimates stale, not after every row write.

## Batch related writes

When an application must insert many rows, a prepared statement batch can reduce repeated JDBC calls. Keep a clear transaction boundary and roll back the transaction if a batch fails:

```java
String sql = "INSERT INTO LIBRARY.BOOKS "
        + "(BOOK_ID, TITLE, PUBLISHED_YEAR, AVAILABLE) "
        + "VALUES (?, ?, ?, ?)";

connection.setAutoCommit(false);
try (PreparedStatement statement = connection.prepareStatement(sql)) {
    for (Book book : books) {
        statement.setInt(1, book.id());
        statement.setString(2, book.title());
        statement.setInt(3, book.publishedYear());
        statement.setBoolean(4, book.available());
        statement.addBatch();
    }

    statement.executeBatch();
    connection.commit();
} catch (SQLException exception) {
    connection.rollback();
    throw exception;
} finally {
    connection.setAutoCommit(true);
}
```

This assumes a `Book` type with the shown accessor methods and a connection that is not shared with another task. Choose a batch size that fits the workload and available memory. A single huge transaction can hold locks and log space for too long. If the batch fails, inspect the `BatchUpdateException` and its chained SQL exceptions before deciding whether to retry.

## Keep transactions short

Transactions hold locks until commit or rollback. Read or write the rows needed for one unit of work, finish promptly, and release locks. Avoid waiting for user input, network calls, or long computations while a transaction is open. When two transactions update the same rows, they may wait or deadlock; access shared tables in a consistent order and handle transaction rollback by restarting the complete unit of work.

Connection and transaction behavior is covered in [Transactions and concurrency](09-transactions-and-concurrency.md). Use the same isolation level only when its guarantees are needed. A more restrictive isolation level can increase waiting between concurrent tasks.

## Reclaim space after large deletes

Deleting rows does not necessarily make the database file smaller. Derby can reuse freed space for later rows, and a table may keep allocated pages after a large delete. Check whether space has accumulated before running a maintenance procedure.

`SYSCS_UTIL.SYSCS_COMPRESS_TABLE` rebuilds a table and its indexes to reclaim unused allocated space. Run it in auto-commit mode after backing up the database:

```sql
CALL SYSCS_UTIL.SYSCS_COMPRESS_TABLE(
    'LIBRARY', 'BOOKS', 1
);
```

The final argument `1` requests sequential compression. It uses less temporary disk space and memory than rebuilding the table and indexes concurrently, but takes longer. Compression takes an exclusive table lock, so schedule it when applications can wait. It also updates index statistics. Do not run it as an automatic response to every delete.

## Keep the runtime and database files consistent

Set `derby.system.home` before Derby starts when the application needs databases stored in a known location. Use one Derby version across the application's classpath or module path. `sysinfo` can help identify duplicate or unexpected Derby libraries.

Shut down embedded databases cleanly and stop Network Server processes through their supported controls. Do not rename, edit, or copy individual files inside a running database directory. Use Derby's backup procedures for a live database and shut it down before making a file-system copy.

Before upgrading a database, create and verify a backup, check that only the intended Derby libraries are loaded, and test the upgrade on a copy. A full database upgrade is a one-way change for older Derby versions: after the upgrade, do not expect an older engine to open that database.

Apache Derby entered read-only retired status on October 10, 2025. Its developers have ended development and bug fixes, and no further releases are planned. These notes are useful for learning SQL and maintaining existing Derby installations. Check the project's status before choosing it for new systems.

## Common mistakes

- Adding indexes without measuring the queries that need them.
- Expecting an index to help a query whose plan still scans the full table.
- Using one very wide composite index for unrelated queries.
- Ignoring added storage and write cost when creating indexes.
- Refreshing statistics after every write instead of after meaningful data changes.
- Keeping a transaction open while doing unrelated work.
- Retrying a failed batch without checking which statements succeeded.
- Expecting a large delete to shrink the database file immediately.
- Running compression while an application needs unrestricted access to the table.
- Upgrading the only database copy before testing the process on a backup.
- Loading multiple Derby versions into the same process.
- Copying or editing files in a database directory while Derby is running.

## Practice

1. Run the filtered books query before and after creating `BOOKS_YEAR_IX`, then compare the runtime plans.
2. Insert enough test books to change the data distribution, refresh the index statistics, and compare the plan again.
3. Insert several books using a JDBC batch in a test database, then force a constraint failure and observe the transaction result.
4. Delete many rows from a disposable table, inspect the database size, then compare sequential compression with ordinary compression.
5. Write down which Derby libraries your application loads and test a planned upgrade against a restored copy.
6. Review the current Derby project status before using the engine in a new system.

## Further reading

- [Derby Tuning Guide](https://db.apache.org/derby/docs/10.17/tuning/)
- [Working with cardinality statistics](https://db.apache.org/derby/docs/10.17/tuning/ctunstats57373.html)
- [SYSCS_UTIL.SYSCS_UPDATE_STATISTICS](https://db.apache.org/derby/docs/10.17/ref/rrefupdatestatsproc.html)
- [SYSCS_UTIL.SYSCS_COMPRESS_TABLE](https://db.apache.org/derby/docs/10.17/ref/rrefaltertablecompress.html)
- [Preparing to upgrade](https://db.apache.org/derby/docs/10.17/devguide/tdevpreupgrade.html)
- [upgrade=true connection attribute](https://db.apache.org/derby/docs/10.17/ref/rrefattribupgrade.html)
- [Apache Derby current status](https://db.apache.org/derby/derby_downloads)

| [Previous: Testing and troubleshooting](16-testing-and-troubleshooting.md) | [Notes index](../README.md) | [Next: All code samples](98-all-code-samples.md) |
| --- | --- | --- |
