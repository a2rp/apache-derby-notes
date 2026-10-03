# 15. Authentication, authorization, and security

[Back to notes index](../README.md)

| [Previous: Import, export, backup, and recovery](14-import-export-backup-and-recovery.md) | [Notes index](../README.md) | [Next: Testing and troubleshooting](16-testing-and-troubleshooting.md) |
| --- | --- | --- |

## Authentication and authorization solve different problems

Authentication checks who is connecting. Authorization checks what that user is allowed to do after connecting. A password can identify a valid user, but it does not decide whether that user may read `LIBRARY.BOOKS`, add a row, or change the schema.

Derby supports NATIVE authentication, LDAP authentication, and user-defined authentication. NATIVE authentication stores users and encrypted passwords in a Derby database. The examples here use NATIVE authentication because it is available without configuring an external directory service.

## Enable NATIVE authentication

For a new local practice database, connect once as the account that will own the database. Create the database as `DBOWNER`, then store credentials for the owner first, followed by the application users:

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

The password strings above are placeholders. Replace them with unique secrets and do not put real passwords in source control, command history, or shared notes. The first user added through `SYSCS_CREATE_USER` must be the database owner. After the users are created, shut down and reboot the database. NATIVE authentication takes effect on the next boot.

```text
CONNECT 'jdbc:derby:securedb;shutdown=true';
```

For an embedded database, Derby reports that the database shut down as an exception from the shutdown connection. That response is expected. On the next connection, provide a valid user and password. Once NATIVE authentication has been enabled this way, it cannot be turned off for that database.

NATIVE authentication also enables Derby's fine-grained SQL authorization. If another authentication provider is used, enable SQL authorization separately for the database before using `GRANT` and `REVOKE`:

```sql
CALL SYSCS_UTIL.SYSCS_SET_DATABASE_PROPERTY(
    'derby.database.sqlAuthorization', 'true'
);
```

Treat this as a permanent security decision. Derby does not allow the SQL authorization property to be turned back off after it has been enabled.

## Connect with a database identity

In `ij`, provide credentials as connection attributes:

```text
CONNECT 'jdbc:derby:securedb;user=DBOWNER;password=replace-with-owner-password';
```

In JDBC, pass the login separately from the URL. Load the values from a protected configuration source supplied by the deployment environment:

```java
Properties credentials = new Properties();
credentials.setProperty("user", System.getenv("DERBY_USER"));
credentials.setProperty("password", System.getenv("DERBY_PASSWORD"));

try (Connection connection = DriverManager.getConnection(
        "jdbc:derby:securedb", credentials)) {
    System.out.println("Connected as " + connection.getMetaData().getUserName());
}
```

For a Network Server connection, the URL names the server and database, for example `jdbc:derby://localhost:1527/securedb`. Keep credentials outside the URL when application configuration allows it. Never print a password or include it in an exception message that may be logged.

## Give users only the needed permissions

After authentication is enabled, use roles to group privileges by job. The database owner can create roles and grant them to users. This example makes a read-only role for the library's books table:

```sql
CONNECT 'jdbc:derby:securedb;user=DBOWNER;password=replace-with-owner-password';

CREATE ROLE library_reader;
GRANT SELECT ON TABLE LIBRARY.BOOKS TO library_reader;
GRANT library_reader TO APP_READER;
```

The reader connects with its own identity, then activates the role for the session:

```sql
CONNECT 'jdbc:derby:securedb;user=APP_READER;password=replace-with-reader-password';
SET ROLE library_reader;

SELECT BOOK_ID, TITLE
FROM LIBRARY.BOOKS;
```

The role cannot insert or delete rows because it has only `SELECT`. A writer can receive only the changes it needs:

```sql
CONNECT 'jdbc:derby:securedb;user=DBOWNER;password=replace-with-owner-password';

CREATE ROLE library_writer;
GRANT SELECT, INSERT, UPDATE ON TABLE LIBRARY.BOOKS TO library_writer;
GRANT library_writer TO APP_WRITER;
```

Avoid `ALL PRIVILEGES` and broad grants to `PUBLIC` when a smaller permission set is enough. `PUBLIC` applies to all current and future users. A user with no privilege on an application table should not be able to query or modify it.

Remove a permission when a role no longer needs it:

```sql
REVOKE INSERT ON TABLE LIBRARY.BOOKS FROM library_writer;
```

The object owner or database owner can grant privileges on a table. The database owner controls grants of roles to users. Derby returns an authorization error when a user attempts an operation that their identity and active roles do not permit.

## Protect the Network Server

Keep the Network Server bound to localhost unless remote access is required. If clients connect across a network, restrict access with a firewall and protect the connection with SSL/TLS. Authentication alone does not encrypt the data sent between a client and server. Derby documents other network authentication mechanisms, but they do not replace transport encryption.

Protect the database directory, backup directories, and `derby.properties` with operating system permissions. Limit who can read or change them, and do not expose the database files through a public file share. A user with operating system access to those files may bypass application-level controls or damage the data.

## Common mistakes

- Treating a valid login as permission to read or change every table.
- Creating another NATIVE user before adding credentials for the database owner.
- Forgetting to shut down and reboot after initially adding NATIVE credentials.
- Assuming NATIVE authentication can be disabled after it has been enabled.
- Turning on SQL authorization without recording that the setting cannot be reversed.
- Giving an application account database-owner credentials.
- Granting `ALL PRIVILEGES` or `PUBLIC` when a limited role is sufficient.
- Putting passwords in source files, public repositories, URLs, or application logs.
- Exposing a Network Server to a network without firewall rules and SSL/TLS.
- Assuming network authentication encrypts query results and other traffic.

## Practice

1. Create an isolated database and enable NATIVE authentication with distinct owner, reader, and writer accounts.
2. Shut down and reconnect with the owner password, then verify that a connection without a password fails.
3. Grant `SELECT` on `LIBRARY.BOOKS` to a reader role and check that an insert is rejected.
4. Grant `SELECT`, `INSERT`, and `UPDATE` to a writer role, then confirm that delete remains unavailable.
5. Revoke one permission and test the operation again in a new session.
6. Review where your application reads its database credentials and confirm that none are committed or written to logs.
7. Review the Network Server host, firewall rules, and TLS configuration before allowing remote clients.

## Further reading

- [Derby Security Guide](https://db.apache.org/derby/docs/10.17/security/)
- [Configuring NATIVE authentication](https://db.apache.org/derby/docs/10.17/security/cseccsecurenativeauth.html)
- [Configuring fine-grained user authorization](https://db.apache.org/derby/docs/10.17/security/csecauthorfine.html)
- [NATIVE authentication and SQL authorization example](https://db.apache.org/derby/docs/10.17/security/rseccsecurenativeauthex.html)
- [SYSCS_UTIL.SYSCS_CREATE_USER](https://db.apache.org/derby/docs/10.17/ref/rrefnativecreateuserproc.html)
- [GRANT statement](https://db.apache.org/derby/docs/10.17/ref/rrefsqljgrant.html)
- [REVOKE statement](https://db.apache.org/derby/docs/10.17/ref/rrefsqljrevoke.html)
- [Network Server security](https://db.apache.org/derby/docs/10.17/security/tseccsecure82556.html)

| [Previous: Import, export, backup, and recovery](14-import-export-backup-and-recovery.md) | [Notes index](../README.md) | [Next: Testing and troubleshooting](16-testing-and-troubleshooting.md) |
| --- | --- | --- |
