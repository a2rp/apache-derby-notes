# 10. JDBC connections and prepared statements

[Back to notes index](../README.md)

| [Previous: Transactions and concurrency](09-transactions-and-concurrency.md) | [Notes index](../README.md) | [Next: Embedded mode and database lifecycle](11-embedded-mode-and-lifecycle.md) |
| --- | --- | --- |

## How JDBC fits together

JDBC is Java's standard API for working with a relational database. A typical read follows this path:

1. `DriverManager` opens a `Connection` using a database URL.
2. The connection creates a `PreparedStatement` containing SQL.
3. The statement runs and returns a `ResultSet`.
4. The application reads each row from the result set.
5. Try-with-resources closes the result set, statement, and connection.

For Derby 10.17 with Java 21, put the Derby libraries on the application's classpath as described in chapter 2. Derby loads its JDBC driver automatically when `DriverManager` first requests a connection, so a separate `Class.forName` call is not needed in this setup.

## Open an embedded connection

An embedded connection uses the database engine in the same Java process:

```java
String url = "jdbc:derby:librarydb";
Connection connection = DriverManager.getConnection(url);
```

Use `create=true` only when the application should create a database that does not exist yet:

```java
String url = "jdbc:derby:librarydb;create=true";
```

Without that attribute, a missing database causes a connection error. Derby resolves the relative name `librarydb` from the Java process's working directory. Chapter 11 covers embedded database ownership and shutdown.

`DriverManager` keeps small examples direct. Applications running inside a server or framework commonly receive a `DataSource` and call its `getConnection()` method instead. Code that depends on the `Connection` interface can work with either source.

## Choose a statement type

Use `Statement` for fixed SQL that has no changing values. Use `PreparedStatement` when a query has input values, when data comes from a user, or when the same SQL runs with different values:

```java
try (java.sql.Statement statement = connection.createStatement();
     ResultSet results = statement.executeQuery(
             "SELECT BOOK_ID, TITLE FROM LIBRARY.BOOKS ORDER BY TITLE")) {
    while (results.next()) {
        System.out.println(results.getString("TITLE"));
    }
}
```

Prepared statements also make parameter types explicit and keep data values separate from SQL text. The next section shows their parameter syntax.

## Read rows with a prepared statement

A `PreparedStatement` keeps SQL structure separate from values. Each `?` is a parameter. Set parameters in order, starting at 1:

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public class FindBooks {
    public static void main(String[] args) throws SQLException {
        String url = "jdbc:derby:librarydb";
        String sql = "SELECT BOOK_ID, TITLE, PUBLISHED_YEAR "
                + "FROM LIBRARY.BOOKS "
                + "WHERE TITLE LIKE ? "
                + "ORDER BY TITLE";

        try (Connection connection = DriverManager.getConnection(url);
             PreparedStatement statement = connection.prepareStatement(sql)) {

            statement.setString(1, "%SQL%");

            try (ResultSet results = statement.executeQuery()) {
                while (results.next()) {
                    int bookId = results.getInt("BOOK_ID");
                    String title = results.getString("TITLE");
                    int yearValue = results.getInt("PUBLISHED_YEAR");
                    Integer publishedYear = results.wasNull()
                            ? null
                            : yearValue;

                    System.out.println(bookId + " | " + title + " | "
                            + publishedYear);
                }
            }
        }
    }
}
```

The result set cursor starts before the first row. Each `next()` call moves it forward and returns `false` when there are no more rows. `getInt` returns `0` for SQL `NULL`, so call `wasNull()` immediately after reading the column when the difference between `NULL` and zero matters.

## Change data and check the row count

Use `executeUpdate()` for `INSERT`, `UPDATE`, and `DELETE`. It returns the number of changed rows:

```java
String sql = "UPDATE LIBRARY.BOOKS "
        + "SET AVAILABLE = ? "
        + "WHERE BOOK_ID = ?";

try (PreparedStatement statement = connection.prepareStatement(sql)) {
    statement.setBoolean(1, false);
    statement.setInt(2, 3);

    int changedRows = statement.executeUpdate();
    System.out.println("Updated rows: " + changedRows);
}
```

When the application expects exactly one row, check that the returned count is `1`. A count of `0` means no row matched. A count above `1` may mean the `WHERE` condition is too broad.

Use `executeQuery()` for a statement that returns rows. Use `executeUpdate()` for a data change or a statement that returns no rows. `execute()` is available when the result type is not known in advance.

## Bind values safely

Do not build SQL by joining user input into a string:

```java
// Avoid this pattern.
String sql = "SELECT BOOK_ID FROM LIBRARY.BOOKS WHERE TITLE = '"
        + userInput + "'";
```

Pass the value as a parameter instead:

```java
String sql = "SELECT BOOK_ID FROM LIBRARY.BOOKS WHERE TITLE = ?";

try (PreparedStatement statement = connection.prepareStatement(sql)) {
    statement.setString(1, userInput);
    try (ResultSet results = statement.executeQuery()) {
        while (results.next()) {
            System.out.println(results.getInt("BOOK_ID"));
        }
    }
}
```

Parameters represent values, not SQL structure. A placeholder cannot stand for a table name, column name, or sort direction. If a query must choose among identifiers, map an application choice to a fixed list of known SQL fragments.

Use a typed setter such as `setInt`, `setBoolean`, `setDate`, or `setString` when the value type is known. Use `setNull(parameterNumber, sqlType)` when setting a SQL `NULL` value.

## Retrieve an identity value

When an insert creates a row with an identity column, request the generated key and read it from the returned result set:

```java
String sql = "INSERT INTO LIBRARY.LOANS "
        + "(BOOK_ID, MEMBER_ID, CHECKED_OUT_ON) "
        + "VALUES (?, ?, CURRENT_DATE)";

try (PreparedStatement statement = connection.prepareStatement(
        sql, java.sql.Statement.RETURN_GENERATED_KEYS)) {
    statement.setInt(1, 3);
    statement.setInt(2, 2);
    statement.executeUpdate();

    try (ResultSet keys = statement.getGeneratedKeys()) {
        if (keys.next()) {
            int loanId = keys.getInt(1);
            System.out.println("Created loan " + loanId);
        }
    }
}
```

Use generated keys for a single-row insert. Derby documents limitations for generated keys with multi-row inserts, so retrieve each identity value through a separate single-row insert when each generated key is needed.

## Close JDBC resources

Connections, statements, and result sets use database and application resources. Try-with-resources closes them even when a query fails. Close nested resources in the order they are used:

```java
try (Connection connection = DriverManager.getConnection(url);
     PreparedStatement statement = connection.prepareStatement(sql);
     ResultSet results = statement.executeQuery()) {
    while (results.next()) {
        // Read the current row here.
    }
}
```

When statements are part of a transaction, use chapter 9's commit and rollback structure. Try-with-resources can still close statements and result sets, while the connection stays open until the transaction finishes.

## Read JDBC errors

JDBC reports database and connection problems with `SQLException`. Inspect the message and SQL state, and follow the chained exceptions when present:

```java
catch (SQLException error) {
    System.err.println("Message: " + error.getMessage());
    System.err.println("SQL state: " + error.getSQLState());
    System.err.println("Error code: " + error.getErrorCode());

    for (SQLException next = error.getNextException();
         next != null;
         next = next.getNextException()) {
        System.err.println("Related error: " + next.getMessage());
    }
}
```

Use SQL states to make application decisions only when the Derby documentation defines the state for that error. Chapter 16 covers common failures and diagnostics.

## Common mistakes

- Forgetting that parameter positions start at `1`, not `0`.
- Calling `executeQuery()` for an update or `executeUpdate()` for a query.
- Leaving a result set or statement open after the operation finishes.
- Reading a nullable numeric column without checking `wasNull()`.
- Concatenating input values into SQL.
- Trying to bind a table name or column name with `?`.
- Reusing a connection after an error without checking its transaction state.
- Expecting `getGeneratedKeys()` to return a key without requesting generated keys first.

## Practice

1. Run `FindBooks` and change the title pattern to search for another word.
2. Change the query to filter on `PUBLISHED_YEAR` with an integer parameter.
3. Update one book and check that the row count is exactly one.
4. Read `PUBLISHED_YEAR` from a row where it is `NULL` and compare `wasNull()` with the numeric result.
5. Insert a loan and retrieve its generated `LOAN_ID`.
6. Temporarily omit one Derby jar from the runtime classpath and record the connection error. Restore the jar before continuing.

## Further reading

- [Derby embedded basics](https://db.apache.org/derby/docs/10.17/devguide/cdevdvlp39409.html)
- [Derby JDBC driver loading](https://db.apache.org/derby/docs/10.17/devguide/cdevdvlp40653.html)
- [DriverManager and Derby connection URLs](https://db.apache.org/derby/docs/10.17/ref/rrefjdbc34565.html)
- [PreparedStatement interface](https://db.apache.org/derby/docs/10.17/ref/rrefjdbc29874.html)
- [ResultSet interface](https://db.apache.org/derby/docs/10.17/ref/rrefjdbc23502.html)
- [Autogenerated keys](https://db.apache.org/derby/docs/10.17/ref/crefjavstateautogen.html)

| [Previous: Transactions and concurrency](09-transactions-and-concurrency.md) | [Notes index](../README.md) | [Next: Embedded mode and database lifecycle](11-embedded-mode-and-lifecycle.md) |
| --- | --- | --- |
