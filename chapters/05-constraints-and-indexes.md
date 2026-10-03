# 5. Constraints and indexes

[Back to notes index](../README.md)

| [Previous: Schemas, tables, and data types](04-schemas-tables-and-data-types.md) | [Notes index](../README.md) | [Next: Inserting, updating, and deleting data](06-insert-update-delete.md) |
| --- | --- | --- |

## Constraints protect data

A constraint is a rule that Derby checks when data is inserted or changed. Put important rules in the database so every client must follow them.

Common constraints include:

- `NOT NULL` requires a value.
- `PRIMARY KEY` uniquely identifies each row and does not allow null values.
- `UNIQUE` prevents duplicate values across one or more columns.
- `CHECK` requires a Boolean condition to be true or unknown for a row.
- `FOREIGN KEY` requires a reference to an existing key in another table.

Use names for important constraints. Clear names make database errors and schema changes easier to understand.

## Primary and unique keys

A primary key is the main identifier for a row. A table can have one primary key, which may contain one or more columns. A unique constraint can protect another candidate identifier.

```sql
CREATE TABLE LIBRARY.PUBLISHERS (
    PUBLISHER_ID INTEGER NOT NULL,
    PUBLISHER_NAME VARCHAR(120) NOT NULL,
    WEBSITE VARCHAR(240),
    CONSTRAINT PUBLISHERS_PK PRIMARY KEY (PUBLISHER_ID),
    CONSTRAINT PUBLISHERS_NAME_UQ UNIQUE (PUBLISHER_NAME)
);
```

For a composite key, the combination of columns must be unique:

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

The referenced key must be a primary key or a unique key. A composite foreign key must use the same number of columns and compatible types in the same order as its referenced key.

## Check constraints and defaults

Use a `CHECK` constraint for a rule that depends on values in the same row:

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

The default value is used only when an insert omits the column or requests `DEFAULT`. It does not replace an explicitly supplied `NULL`, which is why `NOT NULL` is useful with a required default.

`CHECK` does not replace `NOT NULL`. In SQL, a condition involving `NULL` can be unknown, and a check does not reject unknown. Declare required columns as `NOT NULL` as well as adding a check rule.

## Referential integrity

A foreign key keeps child rows connected to a parent row. For example, each copy in `BOOK_COPIES` must reference a book in `BOOKS`.

The default delete behavior prevents deleting a referenced parent row. Choose a referential action only when it matches the application's data ownership rules. `ON DELETE CASCADE` removes dependent rows automatically, so use it only when that deletion is intended.

Add a rule to a table that already exists with `ALTER TABLE`:

```sql
ALTER TABLE LIBRARY.BOOK_COPIES
ADD CONSTRAINT BOOK_COPIES_ID_CK
CHECK (COPY_ID > 0);
```

Before adding a constraint to an existing table, inspect the rows. Derby will reject the change if existing data violates the rule.

## Indexes speed up selected lookups

An index is a data structure Derby can use to find rows without scanning the entire table. Primary key and unique constraints already require supporting indexes, so do not add another index over the same columns without measuring a real need.

```sql
CREATE INDEX BOOKS_TITLE_IX
ON LIBRARY.BOOKS (TITLE);

CREATE INDEX COPIES_BOOK_CONDITION_IX
ON LIBRARY.BOOK_COPIES (BOOK_ID, CONDITION);
```

The order of columns in a composite index matters. An index on `(BOOK_ID, CONDITION)` is useful for searches beginning with `BOOK_ID`; it may not serve a search that filters only on `CONDITION` as well.

Indexes also take disk space and add work to inserts, updates, and deletes. Add indexes for frequent filters, joins, or ordering only after examining the query and its workload.

## Common mistakes

- Treating a `CHECK` rule as a replacement for `NOT NULL`.
- Referencing a parent column that is not a primary key or unique key.
- Adding an index that duplicates the index already created for a primary key or unique constraint.
- Adding many indexes without measuring the cost to writes.
- Assuming a foreign key automatically deletes the parent or child rows.
- Adding a constraint before checking existing data for violations.

## Practice

1. Add a unique constraint for the publisher name and test a duplicate insert.
2. Add a check that limits copy condition values to a short list.
3. Insert a copy with a book ID that does not exist and inspect the foreign-key error.
4. Create a two-column index and write a query whose filter begins with the first indexed column.
5. Explain why a primary key does not need a duplicate manually created index.

## Further reading

- [CREATE TABLE statement](https://db.apache.org/derby/docs/10.17/ref/rrefsqlj24513.html)
- [CREATE INDEX statement](https://db.apache.org/derby/docs/10.17/ref/rrefsqlj20937.html)
- [Derby constraints and referential integrity](https://db.apache.org/derby/docs/10.17/ref/toc.html)

| [Previous: Schemas, tables, and data types](04-schemas-tables-and-data-types.md) | [Notes index](../README.md) | [Next: Inserting, updating, and deleting data](06-insert-update-delete.md) |
| --- | --- | --- |
