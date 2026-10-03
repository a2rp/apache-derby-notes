# 1. Relational databases and Derby

[Back to notes index](../README.md)

| [Previous: Notes index](../README.md) | [Notes index](../README.md) | [Next: Java compatibility, setup, and tools](02-compatibility-setup-and-tools.md) |
| --- | --- | --- |

## What Apache Derby is

Apache Derby is a relational database engine written in Java. An application sends SQL statements to Derby, and Derby stores, reads, and updates structured data. Java applications connect to Derby through JDBC, the standard Java API for working with relational databases.

Derby can run in two common ways:

- **Embedded mode:** the database engine runs inside the Java application process. The application and database share one process and access the database files locally.
- **Network Server mode:** a separate Derby server process owns the database connection. Java clients connect to it over the network with the Derby client JDBC driver.

Choose embedded mode for a single application that owns its local database. Choose Network Server mode when separate processes need to connect to one running database. Do not let two embedded engines open the same database files at the same time.

## How relational data is organized

A database contains schemas. A schema groups objects such as tables, views, indexes, and routines. A table stores rows. Each row has values for the table's columns.

For a small library database:

| Database object | Example | Purpose |
| --- | --- | --- |
| Schema | `APP` | Groups application objects |
| Table | `BOOKS` | Stores one row per book |
| Column | `TITLE` | Names one value in every row |
| Row | `1, 'Clean Code', 2008` | Stores one book record |
| Primary key | `BOOK_ID` | Identifies a row uniquely |

SQL describes the data operation. JDBC carries SQL from Java code to the database and returns results or errors.

## First SQL example

In the `ij` command tool, connect to a local database. The `create=true` attribute creates the database if it does not exist:

```sql
CONNECT 'jdbc:derby:librarydb;create=true';

CREATE TABLE BOOKS (
    BOOK_ID INTEGER NOT NULL PRIMARY KEY,
    TITLE VARCHAR(120) NOT NULL,
    PUBLISHED_YEAR INTEGER
);

INSERT INTO BOOKS (BOOK_ID, TITLE, PUBLISHED_YEAR)
VALUES (1, 'Clean Code', 2008);

SELECT BOOK_ID, TITLE, PUBLISHED_YEAR
FROM BOOKS;
```

The query returns one row:

| BOOK_ID | TITLE | PUBLISHED_YEAR |
| ---: | --- | ---: |
| 1 | Clean Code | 2008 |

The database name in the URL is `librarydb`. Derby creates database files in the current working directory unless the connection URL or system settings specify another location.

## The same connection idea in Java

JDBC uses a connection URL to select the database and driver mode. This short example opens an embedded connection and runs a query:

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;

public class ReadBooks {
    public static void main(String[] args) throws Exception {
        String url = "jdbc:derby:librarydb";

        try (Connection connection = DriverManager.getConnection(url);
             Statement statement = connection.createStatement();
             ResultSet results = statement.executeQuery(
                     "SELECT BOOK_ID, TITLE FROM BOOKS")) {

            while (results.next()) {
                int id = results.getInt("BOOK_ID");
                String title = results.getString("TITLE");
                System.out.println(id + ": " + title);
            }
        }
    }
}
```

The `try` with resources block closes the result set, statement, and connection when the block ends. Later chapters explain these JDBC objects and why closing them matters.

## Common points of confusion

- A Derby **database** is not the same as a SQL **schema**. The database is the stored database unit. A schema is a namespace inside that database.
- `create=true` belongs in the JDBC URL only when the application should create a missing database. Without it, a typo in the path can produce a connection error instead of creating a new database.
- The embedded driver and the network client driver are different. Their JDBC URLs also differ.
- The Java process that uses embedded mode needs the Derby engine libraries on its runtime classpath.
- Derby is retired. Learning its behavior is useful for existing systems, but check current project status and maintenance requirements before selecting it for a new production system.

## Practice

1. In your own words, describe the difference between a database, schema, table, column, and row.
2. Change the example so that it stores two books, then select both rows.
3. Decide whether an embedded connection or a Network Server connection fits an application with one process and one local database.
4. Remove `create=true` from the URL and connect to a database that does not exist. Record the error and explain why it appears.

## Further reading

- [Apache Derby overview and manuals](https://db.apache.org/derby/manuals/)
- [Derby connection URL attributes](https://db.apache.org/derby/docs/10.17/ref/rrefattrib24612.html)
- [Embedded Derby basics](https://db.apache.org/derby/docs/10.17/devguide/cdevdvlp39409.html)

| [Previous: Notes index](../README.md) | [Notes index](../README.md) | [Next: Java compatibility, setup, and tools](02-compatibility-setup-and-tools.md) |
| --- | --- | --- |
