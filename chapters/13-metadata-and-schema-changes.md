# 13. Metadata and schema changes

[Back to notes index](../README.md)

| [Previous: Network Server and client connections](12-network-server-and-client.md) | [Notes index](../README.md) | [Next: Import, export, backup, and recovery](14-import-export-backup-and-recovery.md) |
| --- | --- | --- |

## Read database metadata through JDBC

Metadata describes the database structure: tables, columns, keys, types, and the columns returned by a query. `DatabaseMetaData` is useful when an application needs to inspect the database it has connected to.

List tables and views in the `LIBRARY` schema:

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

Derby does not use catalogs, so pass `null` for the catalog argument. The schema and table arguments are patterns. `%` matches any number of characters, and `_` matches one character. Unquoted Derby names are stored in uppercase, so `LIBRARY` and `BOOKS` are the usual values for this sample schema.

List the columns of `LIBRARY.BOOKS`:

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

The metadata result includes details such as column size, nullability, default value, and whether a column is generated. Use the method documentation for the full list of result columns.

## Inspect keys and query result columns

`DatabaseMetaData` can report the primary key and foreign keys declared on a table:

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

Use `ResultSetMetaData` when the query's result columns are not known until runtime:

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

The column index starts at `1`, just like a prepared statement's parameter index. Prefer column labels when reading result values by name.

## Query Derby's system catalogs

Derby stores database structure in system tables in the `SYS` schema. You can query them while learning how the catalog is organized:

```sql
SELECT s.SCHEMANAME, t.TABLENAME
FROM SYS.SYSTABLES AS t
INNER JOIN SYS.SYSSCHEMAS AS s
    ON s.SCHEMAID = t.SCHEMAID
WHERE s.SCHEMANAME = 'LIBRARY'
ORDER BY t.TABLENAME;
```

System catalogs are owned by Derby. Query them only for inspection; do not insert, update, or delete their rows. For application code, prefer `DatabaseMetaData`, which exposes structure through the JDBC API.

## Add a column safely

`ALTER TABLE` changes an existing table. This adds a required `CATEGORY` column to the sample books table:

```sql
ALTER TABLE LIBRARY.BOOKS
ADD COLUMN CATEGORY VARCHAR(40) DEFAULT 'general' NOT NULL;
```

Derby adds a new column at the end of the row. The default gives existing rows a value, so the new `NOT NULL` requirement can be satisfied. An added `NOT NULL` column needs a default when the table already contains rows.

Add a constraint after checking that existing data satisfies it:

```sql
ALTER TABLE LIBRARY.BOOKS
ADD CONSTRAINT BOOKS_YEAR_CK
CHECK (PUBLISHED_YEAR IS NULL OR PUBLISHED_YEAR >= 1000);
```

Derby validates existing rows when a foreign key or check constraint is added. If any existing row violates the new rule, the statement fails and the constraint is not added.

## Rename or remove a column

Use `RENAME COLUMN` to give a column a new name:

```sql
RENAME COLUMN LIBRARY.BOOKS.CATEGORY TO SECTION;
```

Renaming a column can fail when a view, trigger, check rule, or generated column depends on it. Close any open cursor that reads the column before applying the rename.

Remove a column with `RESTRICT` when you want Derby to reject the change if a dependent object would become invalid:

```sql
ALTER TABLE LIBRARY.BOOKS
DROP COLUMN SECTION RESTRICT;
```

Avoid `CASCADE` for routine practice because it can remove dependent objects. Review the affected views, constraints, indexes, and application queries before dropping a column.

## Plan a schema migration

Treat a schema change as a migration with a clear before and after state:

1. Inspect the current tables, columns, constraints, and data.
2. Back up the database before a change that could remove or reinterpret data.
3. Test the migration against a copy with representative rows.
4. Apply the DDL once, then verify the new metadata and sample queries.
5. Update application code that reads or writes the changed columns.

Derby supports adding and dropping columns, adding and dropping constraints, widening certain character and large-object columns, and changing selected defaults, nullability, and identity properties. Its `ALTER COLUMN` syntax does not allow arbitrary type changes. For a type change that Derby does not support directly, create a replacement column, copy and validate the data, and then plan the old-column removal as a separate migration.

An existing view that uses `SELECT *` does not automatically gain a newly added base-table column. Drop and recreate that view if the new column should appear in its results. Views with explicit column lists keep their declared shape.

## Common mistakes

- Passing a catalog name instead of `null` to Derby metadata methods.
- Using lowercase patterns when metadata expects the stored uppercase names.
- Editing a `SYS` catalog table directly.
- Adding a `NOT NULL` column to populated data without a default.
- Adding a constraint before checking whether existing rows meet it.
- Assuming Derby can alter any column to any other data type in one statement.
- Dropping a column without checking dependent views, keys, indexes, and application code.
- Assuming a view with `SELECT *` updates its result columns when the base table changes.
- Running a one-time migration more than once without checking its current schema state.

## Practice

1. List the tables and views in the `LIBRARY` schema through `DatabaseMetaData`.
2. List every column in `LIBRARY.BOOKS`, including its type and nullability.
3. Read the primary key columns for `LIBRARY.BOOKS` and the imported keys for `LIBRARY.LOANS`.
4. Query `SYS.SYSTABLES` and `SYS.SYSSCHEMAS` to list the sample tables, then describe why application code should use JDBC metadata.
5. Add a category column with a default and verify its value for existing rows.
6. Add a check constraint, attempt to insert a row that violates it, then rename and remove the category column with the shown statements.
7. Create a view with `SELECT *`, add a column to its base table, and inspect why the view result does not gain that column automatically.

## Further reading

- [DatabaseMetaData interface](https://db.apache.org/derby/docs/10.17/ref/rrefjdbc15905.html)
- [ALTER TABLE statement](https://db.apache.org/derby/docs/10.17/ref/rrefsqlj81859.html)
- [RENAME COLUMN statement](https://db.apache.org/derby/docs/10.17/ref/rrefsqljrenamecolumnstatement.html)
- [Derby system tables](https://db.apache.org/derby/docs/10.17/ref/toc.html)
- [Derby Reference Manual](https://db.apache.org/derby/docs/10.17/ref/refderby.pdf)

| [Previous: Network Server and client connections](12-network-server-and-client.md) | [Notes index](../README.md) | [Next: Import, export, backup, and recovery](14-import-export-backup-and-recovery.md) |
| --- | --- | --- |
