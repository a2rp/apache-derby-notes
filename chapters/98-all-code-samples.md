# 98. All code samples

[Back to notes index](../README.md)

| [Previous: Performance and operating considerations](17-performance-and-operations.md) | [Notes index](../README.md) | [Next: Complete questions and answers](99-complete-q-and-a.md) |
| --- | --- | --- |

This appendix gathers every fenced code block from the core chapters in chapter order. The original chapters contain the explanation, setup, and expected behavior for each example. Some blocks are fragments that continue an earlier example, so use the linked source chapter when you need the surrounding context.

## Chapter 1. Relational databases and Derby

[Open source chapter](./01-relational-databases-and-derby.md)

### Example 1

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

### Example 2

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

## Chapter 2. Java compatibility, setup, and tools

[Open source chapter](./02-compatibility-setup-and-tools.md)

### Example 1

```powershell
java --version
javac --version
```

### Example 2

```text
C:\tools\db-derby-10.17.1.0-bin
```

### Example 3

```powershell
$env:DERBY_HOME = 'C:\tools\db-derby-10.17.1.0-bin'
Get-ChildItem "$env:DERBY_HOME\lib"
```

### Example 4

```powershell
$env:CLASSPATH = @(
    "$env:DERBY_HOME\lib\derby.jar",
    "$env:DERBY_HOME\lib\derbytools.jar",
    "$env:DERBY_HOME\lib\derbyshared.jar"
) -join ';'
```

### Example 5

```powershell
java org.apache.derby.tools.sysinfo
java org.apache.derby.tools.ij
```

### Example 6

```powershell
$env:CLASSPATH -split ';'
Test-Path "$env:DERBY_HOME\lib\derby.jar"
Test-Path "$env:DERBY_HOME\lib\derbytools.jar"
java org.apache.derby.tools.sysinfo
```

## Chapter 3. Using ij and creating a database

[Open source chapter](./03-ij-and-first-database.md)

### Example 1

```powershell
java org.apache.derby.tools.ij
```

### Example 2

```sql
CONNECT 'jdbc:derby:librarydb;create=true';
```

### Example 3

```sql
CONNECT 'jdbc:derby:librarydb';
```

### Example 4

```sql
CREATE TABLE BOOKS (
    BOOK_ID INTEGER NOT NULL PRIMARY KEY,
    TITLE VARCHAR(120) NOT NULL,
    PUBLISHED_YEAR INTEGER,
    AVAILABLE BOOLEAN DEFAULT TRUE NOT NULL
);

INSERT INTO BOOKS (BOOK_ID, TITLE, PUBLISHED_YEAR)
VALUES (1, 'Clean Code', 2008);

INSERT INTO BOOKS (BOOK_ID, TITLE, PUBLISHED_YEAR, AVAILABLE)
VALUES (2, 'The Pragmatic Programmer', 1999, FALSE);
```

### Example 5

```sql
SELECT BOOK_ID, TITLE, PUBLISHED_YEAR, AVAILABLE
FROM BOOKS
ORDER BY BOOK_ID;
```

### Example 6

```sql
CONNECT 'jdbc:derby:librarydb;create=true';

CREATE TABLE BOOKS (
    BOOK_ID INTEGER NOT NULL PRIMARY KEY,
    TITLE VARCHAR(120) NOT NULL,
    PUBLISHED_YEAR INTEGER,
    AVAILABLE BOOLEAN DEFAULT TRUE NOT NULL
);

INSERT INTO BOOKS (BOOK_ID, TITLE, PUBLISHED_YEAR)
VALUES (1, 'Clean Code', 2008);

SELECT BOOK_ID, TITLE, PUBLISHED_YEAR, AVAILABLE
FROM BOOKS;
```

### Example 7

```sql
RUN 'C:/derby-work/library.sql';
```

## Chapter 4. Schemas, tables, and data types

[Open source chapter](./04-schemas-tables-and-data-types.md)

### Example 1

```sql
CREATE SCHEMA LIBRARY;

CREATE TABLE LIBRARY.BOOKS (
    BOOK_ID INTEGER NOT NULL PRIMARY KEY,
    TITLE VARCHAR(120) NOT NULL,
    PUBLISHED_YEAR INTEGER,
    AVAILABLE BOOLEAN DEFAULT TRUE NOT NULL
);

INSERT INTO LIBRARY.BOOKS (BOOK_ID, TITLE, PUBLISHED_YEAR, AVAILABLE)
VALUES (1, 'Clean Code', 2008, TRUE),
       (2, 'The Pragmatic Programmer', 1999, FALSE);
```

### Example 2

```sql
SET SCHEMA LIBRARY;

SELECT BOOK_ID, TITLE
FROM BOOKS;
```

### Example 3

```sql
CREATE TABLE BOOKS (BOOK_ID INTEGER NOT NULL);

SELECT book_id
FROM books;
```

### Example 4

```sql
CREATE TABLE "ReadingList" (
    "BookTitle" VARCHAR(120)
);

INSERT INTO "ReadingList" ("BookTitle")
VALUES ('Database Systems');
```

### Example 5

```sql
CREATE TABLE LIBRARY.MEMBERS (
    MEMBER_ID INTEGER NOT NULL PRIMARY KEY,
    FULL_NAME VARCHAR(100) NOT NULL,
    EMAIL VARCHAR(200),
    JOINED_ON DATE DEFAULT CURRENT_DATE NOT NULL
);

INSERT INTO LIBRARY.MEMBERS (MEMBER_ID, FULL_NAME, EMAIL)
VALUES (1, 'Asha Rao', NULL);
```

### Example 6

```sql
SELECT MEMBER_ID, FULL_NAME
FROM LIBRARY.MEMBERS
WHERE EMAIL IS NULL;
```

### Example 7

```sql
CREATE TABLE LIBRARY.BOOKS_WITH_IDENTITY (
    BOOK_ID INTEGER NOT NULL GENERATED ALWAYS AS IDENTITY
        (START WITH 1, INCREMENT BY 1),
    TITLE VARCHAR(120) NOT NULL,
    PRIMARY KEY (BOOK_ID)
);

INSERT INTO LIBRARY.BOOKS_WITH_IDENTITY (TITLE)
VALUES ('Database Systems');
```

### Example 8

```sql
CREATE TABLE LIBRARY.LOANS (
    LOAN_ID INTEGER NOT NULL GENERATED ALWAYS AS IDENTITY,
    BOOK_ID INTEGER NOT NULL,
    MEMBER_ID INTEGER NOT NULL,
    CHECKED_OUT_ON DATE NOT NULL,
    RETURNED_ON DATE,
    CREATED_AT TIMESTAMP DEFAULT CURRENT_TIMESTAMP NOT NULL,
    CONSTRAINT LOANS_PK PRIMARY KEY (LOAN_ID),
    CONSTRAINT LOANS_BOOK_FK FOREIGN KEY (BOOK_ID)
        REFERENCES LIBRARY.BOOKS (BOOK_ID),
    CONSTRAINT LOANS_MEMBER_FK FOREIGN KEY (MEMBER_ID)
        REFERENCES LIBRARY.MEMBERS (MEMBER_ID)
);

VALUES (DATE '2026-10-03', CURRENT_TIMESTAMP);
```

## Chapter 5. Constraints and indexes

[Open source chapter](./05-constraints-and-indexes.md)

### Example 1

```sql
CREATE TABLE LIBRARY.PUBLISHERS (
    PUBLISHER_ID INTEGER NOT NULL,
    PUBLISHER_NAME VARCHAR(120) NOT NULL,
    WEBSITE VARCHAR(240),
    CONSTRAINT PUBLISHERS_PK PRIMARY KEY (PUBLISHER_ID),
    CONSTRAINT PUBLISHERS_NAME_UQ UNIQUE (PUBLISHER_NAME)
);
```

### Example 2

```sql
CREATE TABLE LIBRARY.BOOK_EDITIONS (
    BOOK_ID INTEGER NOT NULL,
    EDITION_NUMBER SMALLINT NOT NULL,
    ISBN VARCHAR(20),
    CONSTRAINT BOOK_EDITIONS_PK PRIMARY KEY (BOOK_ID, EDITION_NUMBER),
    CONSTRAINT BOOK_EDITIONS_ISBN_UQ UNIQUE (ISBN),
    CONSTRAINT BOOK_EDITIONS_BOOK_FK FOREIGN KEY (BOOK_ID)
        REFERENCES LIBRARY.BOOKS (BOOK_ID)
);
```

### Example 3

```sql
CREATE TABLE LIBRARY.BOOK_COPIES (
    COPY_ID INTEGER NOT NULL GENERATED ALWAYS AS IDENTITY,
    BOOK_ID INTEGER NOT NULL,
    CONDITION VARCHAR(20) DEFAULT 'good' NOT NULL,
    ACQUISITION_COST DECIMAL(8, 2) NOT NULL,
    CONSTRAINT BOOK_COPIES_PK PRIMARY KEY (COPY_ID),
    CONSTRAINT BOOK_COPIES_COST_CK CHECK (ACQUISITION_COST >= 0),
    CONSTRAINT BOOK_COPIES_CONDITION_CK
        CHECK (CONDITION IN ('good', 'worn', 'damaged')),
    CONSTRAINT BOOK_COPIES_BOOK_FK FOREIGN KEY (BOOK_ID)
        REFERENCES LIBRARY.BOOKS (BOOK_ID)
);
```

### Example 4

```sql
ALTER TABLE LIBRARY.BOOK_COPIES
ADD CONSTRAINT BOOK_COPIES_ID_CK
CHECK (COPY_ID > 0);
```

### Example 5

```sql
CREATE INDEX BOOKS_TITLE_IX
ON LIBRARY.BOOKS (TITLE);

CREATE INDEX COPIES_BOOK_CONDITION_IX
ON LIBRARY.BOOK_COPIES (BOOK_ID, CONDITION);
```

## Chapter 6. Inserting, updating, and deleting data

[Open source chapter](./06-insert-update-delete.md)

### Example 1

```sql
INSERT INTO LIBRARY.BOOKS (BOOK_ID, TITLE, PUBLISHED_YEAR)
VALUES (3, 'SQL Fundamentals', 2020);
```

### Example 2

```sql
INSERT INTO LIBRARY.BOOKS (BOOK_ID, TITLE)
VALUES (4, 'O''Reilly SQL Guide');
```

### Example 3

```sql
INSERT INTO LIBRARY.BOOKS (BOOK_ID, TITLE, PUBLISHED_YEAR)
VALUES
    (5, 'SQL Fundamentals', 2020),
    (6, 'JDBC in Practice', 2022);
```

### Example 4

```sql
INSERT INTO LIBRARY.BOOKS (BOOK_ID, TITLE, PUBLISHED_YEAR, AVAILABLE)
VALUES (7, 'Derby Field Notes', 2024, DEFAULT);
```

### Example 5

```sql
INSERT INTO LIBRARY.BOOK_COPIES (BOOK_ID, CONDITION, ACQUISITION_COST)
VALUES (1, DEFAULT, 18.50);
```

### Example 6

```sql
UPDATE LIBRARY.BOOKS
SET AVAILABLE = FALSE
WHERE BOOK_ID = 3;
```

### Example 7

```sql
UPDATE LIBRARY.BOOKS
SET TITLE = 'SQL Fundamentals, Second Edition',
    PUBLISHED_YEAR = 2023
WHERE BOOK_ID = 5;
```

### Example 8

```sql
DELETE FROM LIBRARY.BOOKS
WHERE BOOK_ID = 7;
```

### Example 9

```sql
SELECT BOOK_ID, TITLE
FROM LIBRARY.BOOKS
WHERE PUBLISHED_YEAR < 2000;
```

### Example 10

```sql
CREATE TABLE LIBRARY.OLD_BOOKS (
    BOOK_ID INTEGER NOT NULL,
    TITLE VARCHAR(120) NOT NULL,
    PUBLISHED_YEAR INTEGER
);

INSERT INTO LIBRARY.OLD_BOOKS (BOOK_ID, TITLE, PUBLISHED_YEAR)
SELECT BOOK_ID, TITLE, PUBLISHED_YEAR
FROM LIBRARY.BOOKS
WHERE PUBLISHED_YEAR < 2000;
```

## Chapter 7. Selecting and filtering data

[Open source chapter](./07-select-and-filter.md)

### Example 1

```sql
SELECT BOOK_ID, TITLE, PUBLISHED_YEAR
FROM LIBRARY.BOOKS;
```

### Example 2

```sql
SELECT TITLE AS BOOK_TITLE,
       PUBLISHED_YEAR AS YEAR_PUBLISHED
FROM LIBRARY.BOOKS;
```

### Example 3

```sql
SELECT BOOK_ID, TITLE, PUBLISHED_YEAR
FROM LIBRARY.BOOKS
WHERE PUBLISHED_YEAR >= 2000;
```

### Example 4

```sql
SELECT BOOK_ID, TITLE
FROM LIBRARY.BOOKS
WHERE AVAILABLE = TRUE
  AND (PUBLISHED_YEAR >= 2020 OR PUBLISHED_YEAR IS NULL);
```

### Example 5

```sql
SELECT BOOK_ID, TITLE
FROM LIBRARY.BOOKS
WHERE PUBLISHED_YEAR IS NULL;
```

### Example 6

```sql
WHERE PUBLISHED_YEAR = NULL
```

### Example 7

```sql
SELECT BOOK_ID, TITLE
FROM LIBRARY.BOOKS
WHERE TITLE LIKE '%SQL%';
```

### Example 8

```sql
SELECT TITLE, PUBLISHED_YEAR
FROM LIBRARY.BOOKS
WHERE PUBLISHED_YEAR BETWEEN 2000 AND 2025;
```

### Example 9

```sql
SELECT BOOK_ID, TITLE
FROM LIBRARY.BOOKS
WHERE PUBLISHED_YEAR IN (1999, 2008, 2022);
```

### Example 10

```sql
SELECT BOOK_ID, TITLE, PUBLISHED_YEAR
FROM LIBRARY.BOOKS
ORDER BY PUBLISHED_YEAR DESC, TITLE ASC;
```

### Example 11

```sql
SELECT BOOK_ID, TITLE
FROM LIBRARY.BOOKS
ORDER BY TITLE
FETCH FIRST 5 ROWS ONLY;
```

### Example 12

```sql
SELECT DISTINCT PUBLISHED_YEAR
FROM LIBRARY.BOOKS
WHERE PUBLISHED_YEAR IS NOT NULL
ORDER BY PUBLISHED_YEAR;
```

### Example 13

```sql
SELECT TITLE,
       CASE
           WHEN AVAILABLE = TRUE THEN 'Available'
           ELSE 'Checked out'
       END AS AVAILABILITY_LABEL
FROM LIBRARY.BOOKS;
```

### Example 14

```sql
SELECT COUNT(*) AS BOOK_COUNT,
       MIN(PUBLISHED_YEAR) AS OLDEST_YEAR,
       MAX(PUBLISHED_YEAR) AS NEWEST_YEAR
FROM LIBRARY.BOOKS;
```

### Example 15

```sql
SELECT PUBLISHED_YEAR, COUNT(*) AS BOOK_COUNT
FROM LIBRARY.BOOKS
WHERE PUBLISHED_YEAR IS NOT NULL
GROUP BY PUBLISHED_YEAR
HAVING COUNT(*) > 1
ORDER BY PUBLISHED_YEAR;
```

## Chapter 8. Joins, subqueries, and views

[Open source chapter](./08-joins-subqueries-and-views.md)

### Example 1

```sql
INSERT INTO LIBRARY.MEMBERS (MEMBER_ID, FULL_NAME, EMAIL)
VALUES (2, 'Dev Shah', 'dev@example.com');

INSERT INTO LIBRARY.LOANS
    (BOOK_ID, MEMBER_ID, CHECKED_OUT_ON, RETURNED_ON)
VALUES (1, 1, DATE '2026-09-01', NULL),
       (2, 2, DATE '2026-09-03', DATE '2026-09-10');
```

### Example 2

```sql
SELECT b.BOOK_ID,
       b.TITLE,
       m.FULL_NAME,
       l.CHECKED_OUT_ON,
       l.RETURNED_ON
FROM LIBRARY.LOANS AS l
INNER JOIN LIBRARY.BOOKS AS b
    ON b.BOOK_ID = l.BOOK_ID
INNER JOIN LIBRARY.MEMBERS AS m
    ON m.MEMBER_ID = l.MEMBER_ID
ORDER BY l.CHECKED_OUT_ON;
```

### Example 3

```sql
SELECT b.BOOK_ID,
       b.TITLE,
       l.CHECKED_OUT_ON,
       m.FULL_NAME
FROM LIBRARY.BOOKS AS b
LEFT OUTER JOIN LIBRARY.LOANS AS l
    ON l.BOOK_ID = b.BOOK_ID
   AND l.RETURNED_ON IS NULL
LEFT OUTER JOIN LIBRARY.MEMBERS AS m
    ON m.MEMBER_ID = l.MEMBER_ID
ORDER BY b.BOOK_ID;
```

### Example 4

```sql
SELECT b.BOOK_ID, b.TITLE
FROM LIBRARY.BOOKS AS b
WHERE b.BOOK_ID IN (
    SELECT l.BOOK_ID
    FROM LIBRARY.LOANS AS l
)
ORDER BY b.BOOK_ID;
```

### Example 5

```sql
SELECT m.MEMBER_ID, m.FULL_NAME
FROM LIBRARY.MEMBERS AS m
WHERE EXISTS (
    SELECT 1
    FROM LIBRARY.LOANS AS l
    WHERE l.MEMBER_ID = m.MEMBER_ID
      AND l.RETURNED_ON IS NULL
)
ORDER BY m.MEMBER_ID;
```

### Example 6

```sql
SELECT m.MEMBER_ID, m.FULL_NAME
FROM LIBRARY.MEMBERS AS m
WHERE NOT EXISTS (
    SELECT 1
    FROM LIBRARY.LOANS AS l
    WHERE l.MEMBER_ID = m.MEMBER_ID
)
ORDER BY m.MEMBER_ID;
```

### Example 7

```sql
SELECT b.BOOK_ID,
       b.TITLE,
       (
           SELECT COUNT(*)
           FROM LIBRARY.LOANS AS l
           WHERE l.BOOK_ID = b.BOOK_ID
       ) AS LOAN_COUNT
FROM LIBRARY.BOOKS AS b
ORDER BY b.BOOK_ID;
```

### Example 8

```sql
SELECT b.BOOK_ID,
       b.TITLE,
       loan_counts.LOAN_COUNT
FROM LIBRARY.BOOKS AS b
LEFT JOIN (
    SELECT l.BOOK_ID, COUNT(*) AS LOAN_COUNT
    FROM LIBRARY.LOANS AS l
    GROUP BY l.BOOK_ID
) AS loan_counts
    ON loan_counts.BOOK_ID = b.BOOK_ID
ORDER BY b.BOOK_ID;
```

### Example 9

```sql
CREATE VIEW LIBRARY.ACTIVE_LOANS
    (BOOK_TITLE, MEMBER_NAME, CHECKED_OUT_ON)
AS
    SELECT b.TITLE, m.FULL_NAME, l.CHECKED_OUT_ON
    FROM LIBRARY.LOANS AS l
    INNER JOIN LIBRARY.BOOKS AS b
        ON b.BOOK_ID = l.BOOK_ID
    INNER JOIN LIBRARY.MEMBERS AS m
        ON m.MEMBER_ID = l.MEMBER_ID
    WHERE l.RETURNED_ON IS NULL;
```

### Example 10

```sql
SELECT BOOK_TITLE, MEMBER_NAME, CHECKED_OUT_ON
FROM LIBRARY.ACTIVE_LOANS
ORDER BY CHECKED_OUT_ON;
```

## Chapter 9. Transactions and concurrency

[Open source chapter](./09-transactions-and-concurrency.md)

### Example 1

```java
connection.setAutoCommit(false);
```

### Example 2

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

### Example 3

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

### Example 4

```java
connection.setTransactionIsolation(Connection.TRANSACTION_SERIALIZABLE);
```

### Example 5

```sql
SET ISOLATION READ COMMITTED;
```

## Chapter 10. JDBC connections and prepared statements

[Open source chapter](./10-jdbc-connections-and-statements.md)

### Example 1

```java
String url = "jdbc:derby:librarydb";
Connection connection = DriverManager.getConnection(url);
```

### Example 2

```java
String url = "jdbc:derby:librarydb;create=true";
```

### Example 3

```java
try (java.sql.Statement statement = connection.createStatement();
     ResultSet results = statement.executeQuery(
             "SELECT BOOK_ID, TITLE FROM LIBRARY.BOOKS ORDER BY TITLE")) {
    while (results.next()) {
        System.out.println(results.getString("TITLE"));
    }
}
```

### Example 4

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

### Example 5

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

### Example 6

```java
// Avoid this pattern.
String sql = "SELECT BOOK_ID FROM LIBRARY.BOOKS WHERE TITLE = '"
        + userInput + "'";
```

### Example 7

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

### Example 8

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

### Example 9

```java
try (Connection connection = DriverManager.getConnection(url);
     PreparedStatement statement = connection.prepareStatement(sql);
     ResultSet results = statement.executeQuery()) {
    while (results.next()) {
        // Read the current row here.
    }
}
```

### Example 10

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

## Chapter 11. Embedded mode and database lifecycle

[Open source chapter](./11-embedded-mode-and-lifecycle.md)

### Example 1

```text
Java application and Derby engine in one JVM
                    |
                    v
           Local database files
```

### Example 2

```powershell
java -Dderby.system.home=C:/data/derby-system `
     -cp "$env:DERBY_HOME/lib/*;." `
     com.example.LibraryApplication
```

### Example 3

```java
String url = "jdbc:derby:librarydb";
```

### Example 4

```java
String url = "jdbc:derby:C:/data/librarydb";
```

### Example 5

```java
DriverManager.getConnection("jdbc:derby:librarydb;shutdown=true");
```

### Example 6

```java
DriverManager.getConnection("jdbc:derby:;shutdown=true");
```

### Example 7

```java
static void shutdownLibrary() throws SQLException {
    try {
        DriverManager.getConnection("jdbc:derby:librarydb;shutdown=true");
        throw new SQLException("Derby did not report a successful shutdown.");
    } catch (SQLException error) {
        if (!"08006".equals(error.getSQLState())) {
            throw error;
        }
    }
}
```

### Example 8

```java
String url = "jdbc:derby:memory:library-test;create=true";
```

### Example 9

```text
jdbc:derby:memory:library-test;drop=true
```

## Chapter 12. Network Server and client connections

[Open source chapter](./12-network-server-and-client.md)

### Example 1

```text
Java application A -- client driver --+
                                      |
Java application B -- client driver --> Derby Network Server -- database files
                                      |
Java command line -- client driver --+
```

### Example 2

```powershell
$env:DERBY_HOME = 'C:\tools\db-derby-10.17.1.0-bin'
java -Dderby.system.home=C:/data/derby-system `
     -jar "$env:DERBY_HOME/lib/derbyrun.jar" server start
```

### Example 3

```powershell
java -jar "$env:DERBY_HOME/lib/derbyrun.jar" server ping
```

### Example 4

```powershell
$env:CLASSPATH = @(
    "$env:DERBY_HOME\lib\derbyclient.jar",
    "$env:DERBY_HOME\lib\derbyshared.jar",
    '.'
) -join ';'
```

### Example 5

```java
String url = "jdbc:derby://localhost:1527/librarydb";

try (Connection connection = DriverManager.getConnection(url);
     PreparedStatement statement = connection.prepareStatement(
             "SELECT BOOK_ID, TITLE FROM LIBRARY.BOOKS ORDER BY BOOK_ID");
     ResultSet results = statement.executeQuery()) {
    while (results.next()) {
        System.out.println(results.getInt("BOOK_ID") + ": "
                + results.getString("TITLE"));
    }
}
```

### Example 6

```java
String url = "jdbc:derby://localhost:1527/librarydb;create=true";
```

### Example 7

```powershell
$env:CLASSPATH = @(
    "$env:DERBY_HOME\lib\derbyclient.jar",
    "$env:DERBY_HOME\lib\derbyshared.jar",
    "$env:DERBY_HOME\lib\derbytools.jar",
    '.'
) -join ';'
java org.apache.derby.tools.ij
```

### Example 8

```sql
CONNECT 'jdbc:derby://localhost:1527/librarydb';

SELECT BOOK_ID, TITLE
FROM LIBRARY.BOOKS
ORDER BY BOOK_ID;
```

### Example 9

```powershell
java -jar "$env:DERBY_HOME/lib/derbyrun.jar" server shutdown
```

## Chapter 13. Metadata and schema changes

[Open source chapter](./13-metadata-and-schema-changes.md)

### Example 1

```java
DatabaseMetaData metadata = connection.getMetaData();
String[] objectTypes = { "TABLE", "VIEW" };

try (ResultSet tables = metadata.getTables(
        null, "LIBRARY", "%", objectTypes)) {
    while (tables.next()) {
        String schema = tables.getString("TABLE_SCHEM");
        String name = tables.getString("TABLE_NAME");
        String type = tables.getString("TABLE_TYPE");
        System.out.println(schema + "." + name + " (" + type + ")");
    }
}
```

### Example 2

```java
try (ResultSet columns = metadata.getColumns(
        null, "LIBRARY", "BOOKS", "%")) {
    while (columns.next()) {
        System.out.println(
                columns.getString("COLUMN_NAME") + " | "
                        + columns.getString("TYPE_NAME") + " | nullable="
                        + columns.getInt("NULLABLE"));
    }
}
```

### Example 3

```java
try (ResultSet keys = metadata.getPrimaryKeys(
        null, "LIBRARY", "BOOKS")) {
    while (keys.next()) {
        System.out.println(
                keys.getString("COLUMN_NAME") + " is part of "
                        + keys.getString("PK_NAME"));
    }
}

try (ResultSet references = metadata.getImportedKeys(
        null, "LIBRARY", "LOANS")) {
    while (references.next()) {
        System.out.println(
                references.getString("FKCOLUMN_NAME") + " references "
                        + references.getString("PKTABLE_SCHEM") + "."
                        + references.getString("PKTABLE_NAME") + "."
                        + references.getString("PKCOLUMN_NAME"));
    }
}
```

### Example 4

```java
try (PreparedStatement statement = connection.prepareStatement(
        "SELECT BOOK_ID, TITLE, PUBLISHED_YEAR "
                + "FROM LIBRARY.BOOKS");
     ResultSet results = statement.executeQuery()) {

    ResultSetMetaData resultInfo = results.getMetaData();
    for (int position = 1;
         position <= resultInfo.getColumnCount();
         position++) {
        System.out.println(
                resultInfo.getColumnLabel(position) + " | "
                        + resultInfo.getColumnTypeName(position));
    }
}
```

### Example 5

```sql
SELECT s.SCHEMANAME, t.TABLENAME
FROM SYS.SYSTABLES AS t
INNER JOIN SYS.SYSSCHEMAS AS s
    ON s.SCHEMAID = t.SCHEMAID
WHERE s.SCHEMANAME = 'LIBRARY'
ORDER BY t.TABLENAME;
```

### Example 6

```sql
ALTER TABLE LIBRARY.BOOKS
ADD COLUMN CATEGORY VARCHAR(40) DEFAULT 'general' NOT NULL;
```

### Example 7

```sql
ALTER TABLE LIBRARY.BOOKS
ADD CONSTRAINT BOOKS_YEAR_CK
CHECK (PUBLISHED_YEAR IS NULL OR PUBLISHED_YEAR >= 1000);
```

### Example 8

```sql
RENAME COLUMN LIBRARY.BOOKS.CATEGORY TO SECTION;
```

### Example 9

```sql
ALTER TABLE LIBRARY.BOOKS
DROP COLUMN SECTION RESTRICT;
```

## Chapter 14. Import, export, backup, and recovery

[Open source chapter](./14-import-export-backup-and-recovery.md)

### Example 1

```sql
CALL SYSCS_UTIL.SYSCS_EXPORT_TABLE(
    'LIBRARY',
    'BOOKS',
    'C:/derby-transfer/books.del',
    ',',
    '"',
    'UTF-8'
);
```

### Example 2

```text
1,"Clean Code",2008,TRUE
2,"The Pragmatic Programmer",1999,FALSE
```

### Example 3

```sql
CREATE TABLE LIBRARY.BOOKS_IMPORT (
    BOOK_ID INTEGER NOT NULL PRIMARY KEY,
    TITLE VARCHAR(120) NOT NULL,
    PUBLISHED_YEAR INTEGER,
    AVAILABLE BOOLEAN DEFAULT TRUE NOT NULL
);
```

### Example 4

```sql
CALL SYSCS_UTIL.SYSCS_IMPORT_TABLE(
    'LIBRARY',
    'BOOKS_IMPORT',
    'C:/derby-transfer/books.del',
    ',',
    '"',
    'UTF-8',
    0
);

SELECT BOOK_ID, TITLE, PUBLISHED_YEAR, AVAILABLE
FROM LIBRARY.BOOKS_IMPORT
ORDER BY BOOK_ID;
```

### Example 5

```sql
CREATE TABLE LIBRARY.BOOKS_IMPORT_MIN (
    BOOK_ID INTEGER NOT NULL PRIMARY KEY,
    TITLE VARCHAR(120) NOT NULL
);
```

### Example 6

```sql
CALL SYSCS_UTIL.SYSCS_IMPORT_DATA(
    'LIBRARY',
    'BOOKS_IMPORT_MIN',
    'BOOK_ID,TITLE',
    '1,2',
    'C:/derby-transfer/books-with-extra-fields.del',
    ',',
    '"',
    'UTF-8',
    0
);
```

### Example 7

```sql
COMMIT;
CALL SYSCS_UTIL.SYSCS_EXPORT_TABLE(
    'LIBRARY', 'BOOKS',
    'C:/derby-transfer/books-next.del', ',', '"', 'UTF-8'
);
```

### Example 8

```sql
CALL SYSCS_UTIL.SYSCS_BACKUP_DATABASE(
    'C:/derby-backups/librarydb-2026-10-03'
);
```

### Example 9

```text
CONNECT 'jdbc:derby:librarydb;restoreFrom=C:/derby-backups/librarydb-2026-10-03';
```

### Example 10

```text
CONNECT 'jdbc:derby:librarydb;rollForwardRecoveryFrom=C:/derby-backups/librarydb-2026-10-03';
```

## Chapter 15. Authentication, authorization, and security

[Open source chapter](./15-authentication-authorization-and-security.md)

### Example 1

```sql
CONNECT 'jdbc:derby:securedb;create=true;user=DBOWNER';

CALL SYSCS_UTIL.SYSCS_CREATE_USER(
    'DBOWNER', 'replace-with-a-strong-owner-password'
);
CALL SYSCS_UTIL.SYSCS_CREATE_USER(
    'APP_READER', 'replace-with-a-strong-reader-password'
);
CALL SYSCS_UTIL.SYSCS_CREATE_USER(
    'APP_WRITER', 'replace-with-a-strong-writer-password'
);
```

### Example 2

```text
CONNECT 'jdbc:derby:securedb;shutdown=true';
```

### Example 3

```sql
CALL SYSCS_UTIL.SYSCS_SET_DATABASE_PROPERTY(
    'derby.database.sqlAuthorization', 'true'
);
```

### Example 4

```text
CONNECT 'jdbc:derby:securedb;user=DBOWNER;password=replace-with-owner-password';
```

### Example 5

```java
Properties credentials = new Properties();
credentials.setProperty("user", System.getenv("DERBY_USER"));
credentials.setProperty("password", System.getenv("DERBY_PASSWORD"));

try (Connection connection = DriverManager.getConnection(
        "jdbc:derby:securedb", credentials)) {
    System.out.println("Connected as " + connection.getMetaData().getUserName());
}
```

### Example 6

```sql
CONNECT 'jdbc:derby:securedb;user=DBOWNER;password=replace-with-owner-password';

CREATE ROLE library_reader;
GRANT SELECT ON TABLE LIBRARY.BOOKS TO library_reader;
GRANT library_reader TO APP_READER;
```

### Example 7

```sql
CONNECT 'jdbc:derby:securedb;user=APP_READER;password=replace-with-reader-password';
SET ROLE library_reader;

SELECT BOOK_ID, TITLE
FROM LIBRARY.BOOKS;
```

### Example 8

```sql
CONNECT 'jdbc:derby:securedb;user=DBOWNER;password=replace-with-owner-password';

CREATE ROLE library_writer;
GRANT SELECT, INSERT, UPDATE ON TABLE LIBRARY.BOOKS TO library_writer;
GRANT library_writer TO APP_WRITER;
```

### Example 9

```sql
REVOKE INSERT ON TABLE LIBRARY.BOOKS FROM library_writer;
```

## Chapter 16. Testing and troubleshooting

[Open source chapter](./16-testing-and-troubleshooting.md)

### Example 1

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

### Example 2

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

### Example 3

```sql
INSERT INTO APP.TEST_BOOKS (BOOK_ID, TITLE)
VALUES (1, 'Duplicate key');
```

### Example 4

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

### Example 5

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

### Example 6

```sql
VALUES SYSCS_UTIL.SYSCS_CHECK_TABLE('LIBRARY', 'BOOKS');
```

### Example 7

```sql
CALL SYSCS_UTIL.SYSCS_SET_RUNTIMESTATISTICS(1);

SELECT BOOK_ID, TITLE
FROM LIBRARY.BOOKS
WHERE PUBLISHED_YEAR >= 2000;

VALUES SYSCS_UTIL.SYSCS_GET_RUNTIMESTATISTICS();

CALL SYSCS_UTIL.SYSCS_SET_RUNTIMESTATISTICS(0);
```

### Example 8

```text
MaximumDisplayWidth 5000
```

### Example 9

```sql
CALL SYSCS_UTIL.SYSCS_SET_RUNTIMESTATISTICS(1);
CALL SYSCS_UTIL.SYSCS_SET_STATISTICS_TIMING(1);
```

### Example 10

```text
java -jar "${env:DERBY_HOME}/lib/derbyrun.jar" sysinfo
```

### Example 11

```text
java -jar "${env:DERBY_HOME}/lib/derbyrun.jar" dblook -d "jdbc:derby:librarydb"
```

## Chapter 17. Performance and operating considerations

[Open source chapter](./17-performance-and-operations.md)

### Example 1

```sql
SELECT BOOK_ID, TITLE
FROM LIBRARY.BOOKS
WHERE PUBLISHED_YEAR >= 2000
ORDER BY PUBLISHED_YEAR, BOOK_ID;
```

### Example 2

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

### Example 3

```sql
CREATE INDEX LIBRARY.BOOKS_YEAR_IX
ON LIBRARY.BOOKS (PUBLISHED_YEAR);
```

### Example 4

```sql
CREATE INDEX LIBRARY.BOOKS_YEAR_TITLE_IX
ON LIBRARY.BOOKS (PUBLISHED_YEAR, TITLE);
```

### Example 5

```sql
CALL SYSCS_UTIL.SYSCS_UPDATE_STATISTICS(
    'LIBRARY', 'BOOKS', NULL
);
```

### Example 6

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

### Example 7

```sql
CALL SYSCS_UTIL.SYSCS_COMPRESS_TABLE(
    'LIBRARY', 'BOOKS', 1
);
```

| [Previous: Performance and operating considerations](17-performance-and-operations.md) | [Notes index](../README.md) | [Next: Complete questions and answers](99-complete-q-and-a.md) |
| --- | --- | --- |
