# 3. Using ij and creating a database

[Back to notes index](../README.md)

| [Previous: Java compatibility, setup, and tools](02-compatibility-setup-and-tools.md) | [Notes index](../README.md) | [Next: Schemas, tables, and data types](04-schemas-tables-and-data-types.md) |
| --- | --- | --- |

## Start ij

`ij` is Derby's command-line tool for connecting to a database and entering SQL. Complete the setup from the previous chapter, then start it from PowerShell:

```powershell
java org.apache.derby.tools.ij
```

The prompt changes to `ij>`. Commands entered there are interpreted by `ij`, while statements such as `CREATE TABLE` and `SELECT` are sent to Derby as SQL.

## Create a database and connect

At the `ij>` prompt, connect with a database URL:

```sql
CONNECT 'jdbc:derby:librarydb;create=true';
```

The URL uses the embedded driver. `librarydb` is the database name, and `create=true` tells Derby to create it when it does not exist. The database is stored relative to the process's working directory in this example.

Connect again without `create=true` when the database is expected to exist:

```sql
CONNECT 'jdbc:derby:librarydb';
```

This second form avoids silently creating a new database if the name or path is wrong.

## Create a table and insert rows

Run the following statements at the `ij>` prompt. Each statement ends with a semicolon:

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

`BOOK_ID` identifies each row. `TITLE` cannot be missing. `PUBLISHED_YEAR` may be unknown because it has no `NOT NULL` rule. `AVAILABLE` receives `TRUE` when an insert omits it.

## Read the rows

```sql
SELECT BOOK_ID, TITLE, PUBLISHED_YEAR, AVAILABLE
FROM BOOKS
ORDER BY BOOK_ID;
```

The result should contain two rows:

| BOOK_ID | TITLE | PUBLISHED_YEAR | AVAILABLE |
| ---: | --- | ---: | --- |
| 1 | Clean Code | 2008 | true |
| 2 | The Pragmatic Programmer | 1999 | false |

Use column names rather than `SELECT *` when the result is part of application code. Named columns make the query's purpose clear and avoid returning columns the application does not need.

## Save commands in a SQL file

For repeatable work, save SQL statements in a text file such as `C:\derby-work\library.sql`:

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

Start `ij` and run the file:

```sql
RUN 'C:/derby-work/library.sql';
```

Forward slashes make Windows file paths easier to read inside the SQL string. Running the complete file a second time will fail at `CREATE TABLE` because the table already exists. Use a fresh database name for a clean practice run, or deliberately remove and recreate the database after confirming that you no longer need its data.

## Understand the files Derby creates

Derby creates database files and a `derby.log` file in the process's working directory by default. The database directory contains internal files managed by the engine. Do not edit those files directly, and do not copy a live database while Derby is using it. Use the backup and shutdown procedures in later chapters.

Keep the database working directory separate from the Derby installation directory. This makes it easier to identify application data and to back it up.

## Common mistakes

- Forgetting the semicolon leaves `ij` waiting for the rest of the statement.
- A JDBC URL is enclosed in quotes after the `CONNECT` command.
- `RUN` is an `ij` command, not a SQL statement. It loads a file containing SQL and other `ij` commands.
- A duplicate primary key is rejected because each `BOOK_ID` must be unique.
- Repeating `CREATE TABLE` fails after the table already exists.
- A relative database name is resolved from the Java process's working directory, which may differ from the folder containing the SQL file.

## Practice

1. Create a database named `practice_library` and connect to it.
2. Add a `PUBLISHER` column using `ALTER TABLE` and choose a suitable text type.
3. Insert a third book with no publication year. Confirm that Derby returns `NULL` for that column.
4. Run a `SELECT` statement with the columns in a different order and compare the result.
5. Run the SQL file twice and write down which statement fails on the second run.

## Further reading

- [Creating a database and running SQL statements with ij](https://db.apache.org/derby/docs/10.17/getstart/twwdactivity1.html)
- [Using the Derby tools](https://db.apache.org/derby/docs/10.17/getstart/cgsusingtoolsutils.html)
- [Database connection URLs in ij](https://db.apache.org/derby/docs/10.17/tools/ctoolsijtools16011.html)

| [Previous: Java compatibility, setup, and tools](02-compatibility-setup-and-tools.md) | [Notes index](../README.md) | [Next: Schemas, tables, and data types](04-schemas-tables-and-data-types.md) |
| --- | --- | --- |
