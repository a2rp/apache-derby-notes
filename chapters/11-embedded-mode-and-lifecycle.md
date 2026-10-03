# 11. Embedded mode and database lifecycle

[Back to notes index](../README.md)

| [Previous: JDBC connections and prepared statements](10-jdbc-connections-and-statements.md) | [Notes index](../README.md) | [Next: Network Server and client connections](12-network-server-and-client.md) |
| --- | --- | --- |

## What embedded mode means

In embedded mode, the Java application and Derby engine run in the same JVM. The application uses the embedded driver, and Derby reads and writes its database files on the local machine.

```text
Java application and Derby engine in one JVM
                    |
                    v
           Local database files
```

Embedded mode fits a standalone application that owns its database, such as a desktop tool or a local service. It avoids running a separate database server process. A second JVM cannot open the same database files while the embedded engine has them booted. Use Derby Network Server when separate application processes need shared access. Chapter 12 covers that mode.

## Locate the Derby system and database

A Derby system is one running Derby engine, a system directory, and the databases available to that engine. The system directory holds system configuration such as `derby.properties` and the default error log `derby.log`. Each disk database lives in its own subdirectory.

Set `derby.system.home` before starting the application so database paths stay predictable:

```powershell
java -Dderby.system.home=C:/data/derby-system `
     -cp "$env:DERBY_HOME/lib/*;." `
     com.example.LibraryApplication
```

With that system directory selected, this relative URL looks for `librarydb` inside it:

```java
String url = "jdbc:derby:librarydb";
```

An absolute path can identify a database elsewhere on the same machine:

```java
String url = "jdbc:derby:C:/data/librarydb";
```

The `derby.system.home` property is read when Derby starts. Set it before the first connection request. If it is omitted, Derby uses the Java process's current directory as its system directory.

## Understand boot and connection ownership

The first connection boots a database. Derby checks the database and performs recovery when needed. A later connection in the same Derby system can connect to an already booted database.

Derby uses a lock file named `db.lck` to prevent another Derby instance from booting a database that is already in use. Do not delete this file to try to force a database open. Close every connection and perform a clean shutdown before moving or restoring database files.

An embedded database can have several `Connection` objects inside its owning application. Use a separate connection for each independent unit of work. Keep transaction boundaries explicit when a task changes multiple rows, as described in chapter 9.

## Shut down a database cleanly

Closing a `Connection` releases that connection's JDBC resources. It does not shut down the Derby engine. When the application owns the embedded engine, close all statements, result sets, and connections before requesting a shutdown.

Shut down just `librarydb` when other databases in the same Derby system should stay open:

```java
DriverManager.getConnection("jdbc:derby:librarydb;shutdown=true");
```

Shut down the entire embedded Derby system when the application is finished with every database:

```java
DriverManager.getConnection("jdbc:derby:;shutdown=true");
```

Derby reports a successful shutdown with an `SQLException`. For a single database the SQL state is `08006`. For the full embedded system it is `XJ015`. Other SQL states indicate a shutdown problem, so do not treat every exception as success.

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

The example expects the documented single-database shutdown state. If it receives a different state, it passes the error to the caller instead of hiding it.

## Use an in-memory database for temporary work

An in-memory database lives in memory instead of a database directory. It is useful for tests and temporary work where the data can be rebuilt:

```java
String url = "jdbc:derby:memory:library-test;create=true";
```

An in-memory database is not a replacement for a saved disk database. It is lost when its Derby engine or JVM shuts down. For the embedded driver, the `drop=true` URL attribute removes a named in-memory database when the test is finished:

```text
jdbc:derby:memory:library-test;drop=true
```

Derby signals a successful drop with SQL state `08006`, so handle that expected exception the same way as a database shutdown. `drop=true` applies to in-memory databases; Derby rejects it for a disk database.

## Preserve the database files

Derby manages the database directory and transaction log. Do not edit individual files, copy a live database directory, or use file deletion as a shutdown method. Use the backup and restore procedures covered in chapter 14. Keep the system directory in a location with permissions appropriate for the account running the application.

## Common mistakes

- Assuming closing the last connection shuts down the embedded engine.
- Using a relative database URL without knowing the process's system directory.
- Changing `derby.system.home` after Derby has already started.
- Starting another JVM against a database that is already booted.
- Deleting `db.lck` or internal database files to bypass a lock.
- Treating the expected shutdown `SQLException` as an ordinary failure without checking its SQL state.
- Expecting an in-memory database to keep data after the engine exits.
- Copying database files while Derby is still using them.

## Practice

1. Set `derby.system.home` to a dedicated local directory and create `librarydb` there.
2. Print the Java process's current directory and compare it with the chosen Derby system directory.
3. Close a connection, reopen the database, and confirm that closing a connection alone did not shut down Derby.
4. Shut down `librarydb`, then connect again and confirm that Derby boots it on the new connection.
5. Shut down the whole system and check that the application handles SQL state `XJ015` as the expected success signal.
6. Create an in-memory database, add a table, shut down its engine, and confirm that the database contents are gone.
7. Start two Java processes and observe that the second one cannot boot the same disk database while the first is using it. Stop the first cleanly before repeating the test.

## Further reading

- [Derby embedded basics](https://db.apache.org/derby/docs/10.17/devguide/cdevdvlp39409.html)
- [Derby system directory and database locks](https://db.apache.org/derby/docs/10.17/devguide/cdevdvlp27610.html)
- [Embedded shutdown example and SQL states](https://db.apache.org/derby/docs/10.17/getstart/rwwdactivity3.html)
- [shutdown=true attribute](https://db.apache.org/derby/docs/10.17/ref/rrefattrib16471.html)
- [Threading and connection modes](https://db.apache.org/derby/docs/10.17/devguide/rdevconcepts713.html)
- [Derby Developer's Guide](https://db.apache.org/derby/docs/10.17/devguide/derbydev.pdf)

| [Previous: JDBC connections and prepared statements](10-jdbc-connections-and-statements.md) | [Notes index](../README.md) | [Next: Network Server and client connections](12-network-server-and-client.md) |
| --- | --- | --- |
