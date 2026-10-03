# 4. Schemas, tables, and data types

[Back to notes index](../README.md)

| [Previous: Using ij and creating a database](03-ij-and-first-database.md) | [Notes index](../README.md) | [Next: Constraints and indexes](05-constraints-and-indexes.md) |
| --- | --- | --- |

## Database, schema, and table

A Derby database contains schemas. A schema is a namespace for database objects, including tables, indexes, views, and routines. The default schema for a new user is commonly `APP`. Derby also has system schemas whose names begin with `SYS`; application objects should use application-owned schemas.

Create a schema and qualify a table name with it:

```sql
CREATE SCHEMA LIBRARY;

CREATE TABLE LIBRARY.BOOKS (
    BOOK_ID INTEGER NOT NULL PRIMARY KEY,
    TITLE VARCHAR(120) NOT NULL
);
```

The fully qualified table name is `LIBRARY.BOOKS`. You can set the current schema for the connection:

```sql
SET SCHEMA LIBRARY;

SELECT BOOK_ID, TITLE
FROM BOOKS;
```

The connection's current schema affects unqualified names. In shared code, qualifying important objects can make the intended schema clear.

## Identifier rules and capitalization

SQL keywords are case-insensitive. Unquoted SQL identifiers are also case-insensitive and are normalized by Derby. Use simple unquoted names with letters, digits, and underscores to avoid case surprises.

```sql
CREATE TABLE BOOKS (BOOK_ID INTEGER NOT NULL);

SELECT book_id
FROM books;
```

Double quotes create a delimited identifier whose spelling and case are preserved. Single quotes are for text values, not table or column names:

```sql
CREATE TABLE "ReadingList" (
    "BookTitle" VARCHAR(120)
);

INSERT INTO "ReadingList" ("BookTitle")
VALUES ('Database Systems');
```

Because the quoted names preserve case, later statements must use the exact quoted spelling. Use delimited names only when a specific requirement calls for them.

## Choosing a data type

Select a type that describes the value and supports the operations the application needs.

| Data | Common Derby type | Example |
| --- | --- | --- |
| Whole count | `INTEGER` | Number of pages |
| Large whole count | `BIGINT` | Large event counter |
| Exact amount | `DECIMAL(10, 2)` | Price with two decimal places |
| Approximate measurement | `DOUBLE` | Sensor reading |
| Short text with a limit | `VARCHAR(120)` | Book title |
| Fixed-width text | `CHAR(2)` | Country code |
| Large text | `CLOB` | Long description |
| True or false | `BOOLEAN` | Whether a book is available |
| Calendar date | `DATE` | Publication date |
| Date and time | `TIMESTAMP` | Record creation time |
| Binary data | `BLOB` | Binary document content |

Use `DECIMAL` for values that require exact decimal arithmetic, such as prices. `DOUBLE` is an approximate floating point type, so rounding differences are expected. Choose a realistic `VARCHAR` limit instead of using a large text type for every string.

## `NULL`, defaults, and required values

`NULL` means that a value is unknown or absent. It is not the same as zero or an empty string. A column allows `NULL` unless it is declared `NOT NULL`.

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

Use `IS NULL` and `IS NOT NULL` to test missing values. A comparison such as `EMAIL = NULL` does not work as a null check.

```sql
SELECT MEMBER_ID, FULL_NAME
FROM LIBRARY.MEMBERS
WHERE EMAIL IS NULL;
```

## Identity columns

An identity column lets Derby generate sequential numeric values. `GENERATED ALWAYS` prevents an insert from supplying its own value in the ordinary way:

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

The application should not assume that generated values are gap-free. A transaction that fails or rolls back can still consume an identity value.

## Useful date and time values

Use SQL date and timestamp values instead of storing a formatted date as ordinary text:

```sql
CREATE TABLE LIBRARY.LOANS (
    LOAN_ID INTEGER NOT NULL PRIMARY KEY,
    CHECKED_OUT_ON DATE NOT NULL,
    CREATED_AT TIMESTAMP DEFAULT CURRENT_TIMESTAMP NOT NULL
);

INSERT INTO LIBRARY.LOANS (LOAN_ID, CHECKED_OUT_ON)
VALUES (1, DATE '2026-10-03');
```

Typed date values can be compared and sorted as dates. Text dates can be sorted alphabetically, which may not match chronological order when formats differ.

## Common mistakes

- Using `=` to compare a value with `NULL` instead of `IS NULL`.
- Quoting every identifier and then forgetting that quoted names preserve case.
- Choosing `DOUBLE` for money even though exact decimal arithmetic is required.
- Putting a date into a `VARCHAR` column and losing date operations.
- Assuming an identity column returns consecutive values with no gaps.
- Forgetting that an unqualified table name depends on the current schema.

## Practice

1. Create a `LIBRARY.PUBLISHERS` table with an identity primary key and a required publisher name.
2. Add a nullable web address and query for rows where it has no value.
3. Create a price column with an exact decimal type. Insert two prices and order them numerically.
4. Create a table with one quoted mixed-case name, then query it using the exact identifier.
5. Change the current schema and observe how an unqualified table name is resolved.

## Further reading

- [Derby 10.17 Reference Manual contents](https://db.apache.org/derby/docs/10.17/ref/toc.html)
- [CREATE TABLE statement](https://db.apache.org/derby/docs/10.17/ref/rrefsqlj24513.html)
- [Derby SQL language reference](https://db.apache.org/derby/docs/10.17/ref/refderby.pdf)

| [Previous: Using ij and creating a database](03-ij-and-first-database.md) | [Notes index](../README.md) | [Next: Constraints and indexes](05-constraints-and-indexes.md) |
| --- | --- | --- |
