# 14. Import, export, backup, and recovery

[Back to notes index](../README.md)

| [Previous: Metadata and schema changes](13-metadata-and-schema-changes.md) | [Notes index](../README.md) | [Next: Authentication, authorization, and security](15-authentication-authorization-and-security.md) |
| --- | --- | --- |

## Move table data with delimited files

Import and export procedures move rows between a Derby table and a delimited text file. They are useful for small migrations, test data, and exchanging data with other tools. They do not copy a database's table definitions, indexes, constraints, users, or complete transaction history. Use a database backup when you need a full database copy.

Export all rows from the sample books table:

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

The six arguments are schema, table, output file, column delimiter, character delimiter, and character set. Derby uses commas and double quotes when the delimiter arguments are `NULL`. The output file must not already exist. Use a new file name for each export, or move an older file out of the way first. With the Network Server, the path belongs to the server machine, because the server reads and writes the file.

The sample file has one row per book and no heading row, for example:

```text
1,"Clean Code",2008,TRUE
2,"The Pragmatic Programmer",1999,FALSE
```

The fields must match the table's column order and types. Text containing a comma is enclosed by the character delimiter so it stays in one field. Choose a character set supported by the Java runtime and use the same one when importing the file.

## Import into a matching table

An import fills an existing table. Create a separate empty table for the first practice run so that imported primary keys do not collide with rows already in `LIBRARY.BOOKS`:

```sql
CREATE TABLE LIBRARY.BOOKS_IMPORT (
    BOOK_ID INTEGER NOT NULL PRIMARY KEY,
    TITLE VARCHAR(120) NOT NULL,
    PUBLISHED_YEAR INTEGER,
    AVAILABLE BOOLEAN DEFAULT TRUE NOT NULL
);
```

Import every field from the exported file. The final argument `0` selects insert mode, which keeps existing rows and adds imported rows:

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

The table column count, order, and data types must agree with the file. Constraints still apply during import, so bad types, duplicate keys, nulls in required columns, or invalid references can make the operation fail. Importing into a table with foreign keys may require loading referenced rows first.

The final import argument controls what happens to existing rows. `0` means insert the imported rows. Any non-zero value means replace: Derby removes the table's existing rows, then loads the file. Replace does not change the table definition or its indexes. Use it only when deleting every current row is intended.

Use `SYSCS_UTIL.SYSCS_IMPORT_DATA` when the file has extra fields or the target table needs only selected fields. Its argument list includes the target column names and one-based input field positions:

For this example, create a separate target table with only the two columns to be loaded:

```sql
CREATE TABLE LIBRARY.BOOKS_IMPORT_MIN (
    BOOK_ID INTEGER NOT NULL PRIMARY KEY,
    TITLE VARCHAR(120) NOT NULL
);
```

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

When importing a subset, make sure omitted target columns can be filled by defaults or allow `NULL`; otherwise the insert cannot satisfy the table definition.

## Import and export transaction behavior

Derby commits when an import or export procedure succeeds and rolls back when it fails. Complete any pending transaction before calling one of these procedures so the procedure does not commit unrelated work or wait on locks held by the current connection:

```sql
COMMIT;
CALL SYSCS_UTIL.SYSCS_EXPORT_TABLE(
    'LIBRARY', 'BOOKS',
    'C:/derby-transfer/books-next.del', ',', '"', 'UTF-8'
);
```

With authentication and SQL authorization enabled, the caller also needs permission to run the procedure and the relevant `SELECT` or `INSERT` permission on the table. Large objects such as `BLOB` and `CLOB` use Derby's LOB import and export procedures, which store the LOB content in external files. The regular table procedure is not a substitute for those LOB procedures.

## Make an online full backup

An online backup copies the whole database while Derby is running. Use an absolute path so its meaning does not depend on the server process's working directory:

```sql
CALL SYSCS_UTIL.SYSCS_BACKUP_DATABASE(
    'C:/derby-backups/librarydb-2026-10-03'
);
```

The argument is the destination directory for this backup copy. Back up to a new location for each retained copy. The procedure brings the database to a consistent state for copying; writes are blocked while the copy runs, while normal reads can continue. Give the Derby process permission to create and write the destination directory. If the database runs on another machine, this path is on that machine.

A backup should be separate from the live database directory. Protect it with the same care as the database, keep more than one recovery point when required, and copy important backups to another storage location. A successful backup call is not proof that your recovery steps work, so periodically restore a copy in an isolated test location and check the data.

## Make an offline backup

For a small local database, an offline file copy is also possible:

1. Shut down the database cleanly and confirm no Derby process still has it open.
2. Copy the entire database directory, including its log and service subdirectories.
3. Store the copy in a separate backup location.
4. Keep the original database directory untouched until the copy is complete.

Do not copy a live database directory with ordinary file-copy tools. A partial copy can capture data files and logs from different points in time. Use the online backup procedure while the database is running.

## Restore a full backup

The `restoreFrom` connection attribute boots a database from a full backup copy. Stop applications that use the database first. Restoring over a database with the same name under `derby.system.home` replaces that database's full contents, so rehearse this with a disposable database or a separate Derby home before using it on important data.

```text
CONNECT 'jdbc:derby:librarydb;restoreFrom=C:/derby-backups/librarydb-2026-10-03';
```

This is a boot-time connection attribute, not a normal SQL statement. Do not combine `restoreFrom` with `create`, `createFrom`, or `rollForwardRecoveryFrom`. Confirm the chosen backup path and target database name before connecting.

## Understand roll-forward recovery

A full backup restores the database to the point when that copy was made. Roll-forward recovery can replay later transactions, but it requires a full backup plus every needed archived log and the active log. Log archiving must have been enabled before the failure, and all required log files must still be available. It recovers the whole database, not a single table.

The boot-time connection attribute is `rollForwardRecoveryFrom`:

```text
CONNECT 'jdbc:derby:librarydb;rollForwardRecoveryFrom=C:/derby-backups/librarydb-2026-10-03';
```

Use this only with a recovery plan tested against a copy. If log archiving was not enabled or required logs are missing, a full backup restore may be the available recovery point. Do not delete archived logs until the backup and recovery policy says they are no longer needed.

## Common mistakes

- Exporting to a file that already exists.
- Using a client machine path when the Network Server needs a server-side path.
- Importing rows into a table whose columns or constraints do not match the file.
- Choosing replace mode when the table's existing rows must be kept.
- Calling import or export with unrelated work still pending in the transaction.
- Copying a live database directory with normal file tools.
- Restoring over the wrong database name or directory.
- Expecting roll-forward recovery to work without archived and active log files.
- Treating an exported delimited file as a full database backup.

## Practice

1. Export `LIBRARY.BOOKS` to a new delimited file and inspect the field order and quoting.
2. Create `LIBRARY.BOOKS_IMPORT` with the matching columns and import the file in insert mode.
3. Query both tables and compare row counts and values.
4. Try importing into a table containing a duplicate primary key and observe the error.
5. Create a new backup directory with the online backup procedure, then restore it under an isolated Derby home.
6. Write down which files and configuration a future roll-forward recovery would require, and explain why a plain export file is insufficient.

## Further reading

- [SYSCS_UTIL.SYSCS_EXPORT_TABLE](https://db.apache.org/derby/docs/10.17/ref/rrefexportproc.html)
- [SYSCS_UTIL.SYSCS_IMPORT_TABLE](https://db.apache.org/derby/docs/10.17/ref/rrefimportproc.html)
- [SYSCS_UTIL.SYSCS_IMPORT_DATA](https://db.apache.org/derby/docs/10.17/ref/rrefimportdataproc.html)
- [SYSCS_UTIL.SYSCS_BACKUP_DATABASE](https://db.apache.org/derby/docs/10.17/ref/rrefbackupdbproc.html)
- [restoreFrom connection attribute](https://db.apache.org/derby/docs/10.17/ref/rrefrestorefrom.html)
- [rollForwardRecoveryFrom connection attribute](https://db.apache.org/derby/docs/10.17/ref/rrefrollforward.html)
- [Backing up and restoring databases](https://db.apache.org/derby/docs/10.17/adminguide/cadminhubbkup98797.html)

| [Previous: Metadata and schema changes](13-metadata-and-schema-changes.md) | [Notes index](../README.md) | [Next: Authentication, authorization, and security](15-authentication-authorization-and-security.md) |
| --- | --- | --- |
