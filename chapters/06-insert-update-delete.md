# 6. Inserting, updating, and deleting data

[Back to notes index](../README.md)

| [Previous: Constraints and indexes](05-constraints-and-indexes.md) | [Notes index](../README.md) | [Next: Selecting and filtering data](07-select-and-filter.md) |
| --- | --- | --- |

## Insert one row

An `INSERT` statement adds a row to a table. List the target columns so the statement remains clear if the table later gains another column:

```sql
INSERT INTO LIBRARY.BOOKS (BOOK_ID, TITLE, PUBLISHED_YEAR)
VALUES (3, 'SQL Fundamentals', 2020);
```

The `AVAILABLE` column receives its default value because it was omitted. The `BOOK_ID`, `TITLE`, and `PUBLISHED_YEAR` values match the table's column types.

Text values use single quotes. To store an apostrophe inside a text value, write it twice:

```sql
INSERT INTO LIBRARY.BOOKS (BOOK_ID, TITLE)
VALUES (4, 'O''Reilly SQL Guide');
```

## Insert more than one row

Derby accepts multiple value groups in one `INSERT` statement:

```sql
INSERT INTO LIBRARY.BOOKS (BOOK_ID, TITLE, PUBLISHED_YEAR)
VALUES
    (5, 'SQL Fundamentals', 2020),
    (6, 'JDBC in Practice', 2022);
```

The whole statement must satisfy table constraints. If a primary key is duplicated or a required value is missing, Derby reports an error.

## Use default and identity values

An insert can explicitly request a default for a column:

```sql
INSERT INTO LIBRARY.BOOKS (BOOK_ID, TITLE, PUBLISHED_YEAR, AVAILABLE)
VALUES (7, 'Derby Field Notes', 2024, DEFAULT);
```

For an identity column, omit the generated column from the insert:

```sql
INSERT INTO LIBRARY.BOOK_COPIES (BOOK_ID, CONDITION, ACQUISITION_COST)
VALUES (1, DEFAULT, 18.50);
```

Derby generates `COPY_ID`, uses the default condition, and stores the supplied cost. The foreign key still requires the referenced book to exist.

## Update selected rows

`UPDATE` changes existing rows. Use a `WHERE` condition to identify exactly which rows should change:

```sql
UPDATE LIBRARY.BOOKS
SET AVAILABLE = FALSE
WHERE BOOK_ID = 3;
```

You can update more than one column in the same statement:

```sql
UPDATE LIBRARY.BOOKS
SET TITLE = 'SQL Fundamentals, Second Edition',
    PUBLISHED_YEAR = 2023
WHERE BOOK_ID = 5;
```

In `ij`, Derby reports how many rows were updated. If the count is zero, the condition matched no rows. If it is more than one, check that the condition selects the intended records.

## Delete selected rows

`DELETE` removes rows that match a condition:

```sql
DELETE FROM LIBRARY.BOOKS
WHERE BOOK_ID = 7;
```

A delete can fail if another table has a foreign key referencing that row. Decide how related data should be handled before deleting a parent record.

Always inspect a condition before using it in an update or delete:

```sql
SELECT BOOK_ID, TITLE
FROM LIBRARY.BOOKS
WHERE PUBLISHED_YEAR < 2000;
```

After confirming the result set, use the same condition in a change statement if that is the intended operation.

## Insert from a query

`INSERT ... SELECT` copies selected values from one table into another. The selected columns must match the target columns in count and compatible type:

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

The destination must not receive duplicate primary keys or rows that violate its constraints. Test the `SELECT` by itself before using it as an insert source.

## Know what each statement changes

| Statement | Purpose | Check before running |
| --- | --- | --- |
| `INSERT` | Adds rows | Required values, types, keys, and defaults |
| `UPDATE` | Changes rows | The `WHERE` condition and expected row count |
| `DELETE` | Removes rows | The rows selected and any foreign key relationships |

An `UPDATE` or `DELETE` without a `WHERE` condition applies to every row in the table. That can be intentional, but it should be a deliberate decision.

## Common mistakes

- Leaving out the target column list and depending on the table's column order.
- Supplying a text value without single quotes.
- Forgetting to double an apostrophe inside a text literal.
- Updating or deleting without checking how many rows match.
- Sending `NULL` when a column is `NOT NULL` or has a needed default.
- Inserting a child row before its referenced parent row exists.
- Reusing a primary key that is already present.

## Practice

1. Add two books with one multiple-row `INSERT` statement.
2. Mark one book unavailable, then select it to confirm the change.
3. Update a title that contains an apostrophe.
4. Copy older book rows into `OLD_BOOKS` and compare the source and destination counts.
5. Write a `DELETE` statement, first run its matching `SELECT`, and record the expected row count.

## Further reading

- [INSERT statement](https://db.apache.org/derby/docs/10.17/ref/rrefsqlj40774.html)
- [UPDATE statement](https://db.apache.org/derby/docs/10.17/ref/rrefsqlj26498.html)
- [DELETE statement](https://db.apache.org/derby/docs/10.17/ref/rrefsqlj35981.html)

| [Previous: Constraints and indexes](05-constraints-and-indexes.md) | [Notes index](../README.md) | [Next: Selecting and filtering data](07-select-and-filter.md) |
| --- | --- | --- |
