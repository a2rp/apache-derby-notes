# 12. Network Server and client connections

[Back to notes index](../README.md)

| [Previous: Embedded mode and database lifecycle](11-embedded-mode-and-lifecycle.md) | [Notes index](../README.md) | [Next: Metadata and schema changes](13-metadata-and-schema-changes.md) |
| --- | --- | --- |

## When to use Network Server

The Derby Network Server runs the database engine in one server process. Applications in other JVMs connect to it over the network with Derby's client JDBC driver.

```text
Java application A -- client driver --+
                                      |
Java application B -- client driver --> Derby Network Server -- database files
                                      |
Java command line -- client driver --+
```

Use this arrangement when multiple applications need concurrent access to one database. The server process owns the database files. Client applications send JDBC requests and do not open those files directly. Only one server process should own a given disk database at a time.

The server listens on localhost by default and uses port `1527`. Localhost is useful for development on one machine. Chapter 15 covers authentication, authorization, and protected connections before remote clients are allowed.

## Start the Network Server on Windows

Set `DERBY_HOME` to the Derby installation folder. Start the server from a dedicated PowerShell window and leave it running while clients connect:

```powershell
$env:DERBY_HOME = 'C:\tools\db-derby-10.17.1.0-bin'
java -Dderby.system.home=C:/data/derby-system `
     -jar "$env:DERBY_HOME/lib/derbyrun.jar" server start
```

The server command reports that it is ready to accept connections on port `1527`. Setting `derby.system.home` on the server process tells it where to find `librarydb`. If the property is omitted, the server uses its current directory as the Derby system directory.

Check that the server is responding from a second PowerShell window:

```powershell
java -jar "$env:DERBY_HOME/lib/derbyrun.jar" server ping
```

Use the same Derby installation for the server and client libraries while learning. For another port or host binding, consult the server guide and apply the security requirements from chapter 15 before allowing connections outside localhost.

## Connect from a Java application

The Network Client driver uses `derbyclient.jar` and the shared Derby library. Add them and the application classes to the client classpath:

```powershell
$env:CLASSPATH = @(
    "$env:DERBY_HOME\lib\derbyclient.jar",
    "$env:DERBY_HOME\lib\derbyshared.jar",
    '.'
) -join ';'
```

Use the client URL format `jdbc:derby://host:port/databaseName`:

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

The `Connection`, `PreparedStatement`, and `ResultSet` APIs are the same as in embedded mode. The URL selects the client driver because it begins with `jdbc:derby://`.

Add `;create=true` only when the client should ask the server to create a missing database:

```java
String url = "jdbc:derby://localhost:1527/librarydb;create=true";
```

For normal use, omit `create=true`. This makes a wrong database name fail clearly instead of creating a new empty database on the server.

## Connect with ij

The `ij` tool can connect through the Network Client too. Start it with the Derby tools available, then use the network URL:

```powershell
$env:CLASSPATH = @(
    "$env:DERBY_HOME\lib\derbyclient.jar",
    "$env:DERBY_HOME\lib\derbyshared.jar",
    "$env:DERBY_HOME\lib\derbytools.jar",
    '.'
) -join ';'
java org.apache.derby.tools.ij
```

```sql
CONNECT 'jdbc:derby://localhost:1527/librarydb';

SELECT BOOK_ID, TITLE
FROM LIBRARY.BOOKS
ORDER BY BOOK_ID;
```

The server process must be running before this connection is made. Close the `ij` connection with `DISCONNECT` when finished.

## Stop the server cleanly

Close client connections before stopping the server. In another PowerShell window on the server machine, run:

```powershell
java -jar "$env:DERBY_HOME/lib/derbyrun.jar" server shutdown
```

The shutdown command contacts the local Network Server and asks it to stop. Use the same port and host options used at startup if the server does not use the defaults. Authentication can require credentials for management commands. Keep the server window open until it confirms shutdown.

## Understand which process owns each file

| Component | Libraries | Responsibility |
| --- | --- | --- |
| Network Server process | `derbynet.jar`, `derby.jar`, `derbyshared.jar` | Opens and manages the database files. |
| Client application | `derbyclient.jar`, `derbyshared.jar` | Sends JDBC requests to the server. |
| `ij` client | `derbytools.jar` and `derbyclient.jar` | Provides a command prompt and connects to the server. |

The client and server must agree on the database name and network address. The server's working directory or `derby.system.home` determines where a relative database name is resolved. A client working directory does not determine the server's database file path.

## Common connection problems

| Symptom | Check |
| --- | --- |
| Connection refused | Confirm the server is running and listening on the requested host and port. |
| Database not found | Check the server's `derby.system.home` and database name. |
| `ClassNotFoundException` for a client class | Add `derbyclient.jar` and `derbyshared.jar` to the client classpath. |
| Client attempts embedded mode | Confirm the URL starts with `jdbc:derby://`. |
| Client and server report different behavior | Confirm both use compatible Derby libraries from the same release family. |
| Server cannot shut down | Run the command on the server machine, use the correct port, and supply required credentials. |

## Common mistakes

- Starting the Network Server and assuming it is the same as an embedded connection.
- Adding `derby.jar` to a remote client when the client driver jar is required.
- Using `jdbc:derby:librarydb` from a remote client instead of the network URL.
- Starting the server from an unexpected system directory.
- Letting an application use `create=true` when the database should already exist.
- Ending the server process without using its shutdown command.
- Binding to a network-facing interface before configuring access controls and transport security.
- Starting multiple servers that try to own the same database files.

## Practice

1. Start the server and confirm that it is ready on localhost port `1527`.
2. Connect with `ij` through `jdbc:derby://localhost:1527/librarydb` and select the sample books.
3. Run the Java query from this chapter using the client classpath.
4. Open a second client process and read the same database through the server.
5. Change the URL to a database name that does not exist and observe the error without `create=true`.
6. Stop the server with its shutdown command, then ping it and confirm it is no longer available.
7. Check the server guide before choosing a non-default host or port.

## Further reading

- [Derby Network Client activity](https://db.apache.org/derby/docs/10.17/getstart/twwdactivity4.html)
- [Database connection URL formats](https://db.apache.org/derby/docs/10.17/tools/ctoolsijtools16011.html)
- [NetworkServerControl API](https://db.apache.org/derby/docs/10.17/publishedapi/org.apache.derby.server/org/apache/derby/drda/NetworkServerControl.html)
- [Network Server security guidance](https://db.apache.org/derby/docs/10.17/security/tseccsecure82556.html)
- [Derby manuals](https://db.apache.org/derby/manuals/)

| [Previous: Embedded mode and database lifecycle](11-embedded-mode-and-lifecycle.md) | [Notes index](../README.md) | [Next: Metadata and schema changes](13-metadata-and-schema-changes.md) |
| --- | --- | --- |
