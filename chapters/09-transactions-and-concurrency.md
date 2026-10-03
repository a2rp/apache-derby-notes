# 9. Transactions and concurrency

[Back to notes index](../README.md)

| [Previous: Joins, subqueries, and views](08-joins-subqueries-and-views.md) | [Notes index](../README.md) | [Next: JDBC connections and prepared statements](10-jdbc-connections-and-statements.md) |
| --- | --- | --- |

## Group related changes into a transaction

A transaction is a unit of database work. It groups statements so the application can make all of them permanent together or undo them together.

The ACID properties describe common transaction goals:

- **Atomicity:** the complete unit succeeds, or its changes are undone.
- **Consistency:** committed changes preserve database rules such as keys and constraints.
- **Isolation:** concurrent work follows the selected isolation level.
- **Durability:** committed changes survive a normal database restart.

In the library example, lending a book involves changing its availability and inserting a loan row. Those two changes belong in one transaction. If either change fails, roll back both.

## Understand auto-commit

A new JDBC connection uses auto-commit by default. Each completed statement is committed as its own transaction. Turn auto-commit off when several statements must succeed together:

```java
connection.setAutoCommit(false);
```

Call `commit()` after all statements succeed. Call `rollback()` after a failure. Both methods end the current transaction and begin a new one on the same connection. There is no separate SQL `BEGIN` command required for ordinary JDBC work.

## Lend a book safely with JDBC

The following complete class reserves book `3` for member `2`. Chapter 6 inserts book `3`, and chapter 8 adds member `2`. Derby updates the book only when it is currently marked available. If no row changes, the transaction is rolled back and no loan is added.

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.SQLException;

public class BorrowBookTransaction {
    public static void main(String[] args) throws SQLException {
        String url = "jdbc:derby:librarydb";

        try (Connection connection = DriverManager.getConnection(url)) {
            connection.setTransactionIsolation(
                    Connection.TRANSACTION_READ_COMMITTED);
            connection.setAutoCommit(false);

            try {
                int bookId = 3;
                int memberId = 2;

                try (PreparedStatement updateBook = connection.prepareStatement(
                        "UPDATE LIBRARY.BOOKS "
                                + "SET AVAILABLE = FALSE "
                                + "WHERE BOOK_ID = ? AND AVAILABLE = TRUE")) {
                    updateBook.setInt(1, bookId);

                    int changedRows = updateBook.executeUpdate();
                    if (changedRows != 1) {
                        throw new SQLException("Book is not available.");
                    }
                }

                try (PreparedStatement insertLoan = connection.prepareStatement(
                        "INSERT INTO LIBRARY.LOANS "
                                + "(BOOK_ID, MEMBER_ID, CHECKED_OUT_ON) "
                                + "VALUES (?, ?, CURRENT_DATE)")) {
                    insertLoan.setInt(1, bookId);
                    insertLoan.setInt(2, memberId);
                    insertLoan.executeUpdate();
                }

                connection.commit();
            } catch (SQLException error) {
                try {
                    connection.rollback();
                } catch (SQLException rollbackError) {
                    error.addSuppressed(rollbackError);
                }
                throw error;
            }
        }
    }
}
```

The `UPDATE` row count is checked before inserting a loan. This protects against a missing book or one already marked unavailable. The foreign keys provide another check that the book and member exist. The outer `try` closes the connection even when an exception occurs.

## Roll back work that should not be kept

Use `rollback()` when a later step fails validation or a statement throws an error. You can also roll back deliberately during practice:

```java
connection.setAutoCommit(false);

try (PreparedStatement statement = connection.prepareStatement(
        "UPDATE LIBRARY.MEMBERS SET EMAIL = ? WHERE MEMBER_ID = ?")) {
    statement.setString(1, "temporary@example.com");
    statement.setInt(2, 1);
    statement.executeUpdate();
}

connection.rollback();
```

After rollback, the email change is not committed. If auto-commit is disabled, always finish the transaction with either `commit()` or `rollback()` before returning the connection to its owner.

## Choose an isolation level

Isolation controls what a transaction can observe while other transactions are running. Derby's default is `READ_COMMITTED` (`CS`, or cursor stability). The JDBC constants map to Derby's names as follows:

| JDBC level | Derby name | General effect |
| --- | --- | --- |
| `TRANSACTION_READ_UNCOMMITTED` | `UR` | Allows dirty reads, so a query can observe another transaction's uncommitted change. |
| `TRANSACTION_READ_COMMITTED` | `CS` | Does not expose uncommitted changes. This is Derby's default. |
| `TRANSACTION_REPEATABLE_READ` | `RS` | Keeps rows read by the transaction stable for repeated reads. |
| `TRANSACTION_SERIALIZABLE` | `RR` | Provides the strongest isolation and may block more concurrent work. |

Set the level before doing transaction work:

```java
connection.setTransactionIsolation(Connection.TRANSACTION_SERIALIZABLE);
```

Changing the isolation level commits the current transaction in Derby. Choose the level before making changes you may still need to roll back. Stronger isolation can reduce concurrency, so keep transactions short and use the level the application requires.

The equivalent command at the `ij` prompt is:

```sql
SET ISOLATION READ COMMITTED;
```

`SET ISOLATION` is a Derby SQL statement. It commits the current transaction when issued, so do not use it in the middle of work that must remain uncommitted.

## Handle concurrent updates

When two connections update the same data, Derby coordinates them with locks. One transaction may wait for another to finish. A deadlock can happen when transactions each hold a lock the other needs. Derby aborts a transaction in a deadlock so the application can recover.

To reduce lock waits and deadlocks:

- Keep transactions short and commit promptly.
- Access shared tables and rows in a consistent order.
- Avoid waiting for user input or network calls while a transaction is open.
- Read the SQL state and exception details before deciding whether retry is appropriate.
- Retry a complete, idempotent unit of work, not just the statement that failed.

## Common mistakes

- Leaving auto-commit on when multiple statements must succeed together.
- Forgetting to commit or roll back before a connection is reused.
- Catching an exception and continuing as if the transaction succeeded.
- Changing isolation after making edits that should remain rollbackable.
- Keeping a transaction open while doing unrelated application work.
- Retrying a write without checking whether repeating it would create duplicate effects.

## Practice

1. Run the loan transaction and confirm that book `3` becomes unavailable and one loan row is added.
2. Run it again and confirm that the availability check prevents another loan row.
3. Make the member ID invalid, run the transaction, and confirm that the book update is rolled back when the foreign key rejects the loan.
4. Update a member email with auto-commit disabled, roll back, then select the row to confirm that the original value remains.
5. Open two connections and update the same book. Record when one connection waits and what happens after the other commits.
6. Compare the result of a read under `READ_COMMITTED` and `SERIALIZABLE` while another connection changes the same row.

## Further reading

- [Derby transaction isolation levels](https://db.apache.org/derby/docs/10.17/ref/rrefjavcsti.html)
- [SET ISOLATION statement](https://db.apache.org/derby/docs/10.17/ref/rrefsqlj41180.html)
- [Derby Developer's Guide](https://db.apache.org/derby/docs/10.17/devguide/derbydev.pdf)
- [Derby manuals](https://db.apache.org/derby/manuals/)

| [Previous: Joins, subqueries, and views](08-joins-subqueries-and-views.md) | [Notes index](../README.md) | [Next: JDBC connections and prepared statements](10-jdbc-connections-and-statements.md) |
| --- | --- | --- |
