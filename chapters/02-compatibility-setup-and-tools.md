# 2. Java compatibility, setup, and tools

[Back to notes index](../README.md)

| [Previous: Relational databases and Derby](01-relational-databases-and-derby.md) | [Notes index](../README.md) | [Next: Using ij and creating a database](03-ij-and-first-database.md) |
| --- | --- | --- |

## Pick a compatible Java and Derby version

Derby releases have different Java requirements. The official download table lists Derby 10.17.1.0 for Java 21 and newer. These notes use that release family for commands and examples. Do not mix Derby libraries from different releases in one application.

Derby was retired to read-only status on October 10, 2025. The project says that development and bug fixes have ended. The published distributions remain available, but no new release should be expected. Check the official downloads page for the current compatibility details and follow the security policy of the system where Derby will run.

## Install a JDK

The JDK provides the Java runtime and tools used to compile Java examples. Open PowerShell and verify the installed version:

```powershell
java --version
javac --version
```

For Derby 10.17.1.0, use a Java 21 or newer JDK. If `java` or `javac` is not found, install a JDK and configure the Windows `PATH` to include its `bin` directory. Open a new terminal after changing environment variables.

## Download and unpack Derby

Download the binary distribution from the official Derby downloads page. Verify the archive with the signature or checksum published beside it before using it. Unpack the archive to a stable path, for example:

```text
C:\tools\db-derby-10.17.1.0-bin
```

Set a PowerShell variable for the current terminal:

```powershell
$env:DERBY_HOME = 'C:\tools\db-derby-10.17.1.0-bin'
Get-ChildItem "$env:DERBY_HOME\lib"
```

The `lib` directory contains the Derby libraries. Common files include `derby.jar` for the embedded engine, `derbytools.jar` for tools such as `ij`, `derbyshared.jar` for shared classes, `derbynet.jar` for the Network Server, and `derbyclient.jar` for network clients.

## Put the tools on the classpath

The classpath tells Java where to find classes. For the embedded engine and the `ij` tool, set it for the current PowerShell session:

```powershell
$env:CLASSPATH = @(
    "$env:DERBY_HOME\lib\derby.jar",
    "$env:DERBY_HOME\lib\derbytools.jar",
    "$env:DERBY_HOME\lib\derbyshared.jar"
) -join ';'
```

Start `ij` and inspect the Derby runtime:

```powershell
java org.apache.derby.tools.sysinfo
java org.apache.derby.tools.ij
```

`ij` is a command-line SQL client. It accepts Derby connection URLs and SQL statements. `sysinfo` prints details about the Java runtime and Derby libraries that Java can find.

For a Network Server setup, the server process also needs `derbynet.jar`. A Java application using a remote client needs `derbyclient.jar`. Keep server and client libraries from the same Derby release family.

## Check which libraries Java found

If a class cannot be found, print the classpath from the same terminal that starts Java:

```powershell
$env:CLASSPATH -split ';'
Test-Path "$env:DERBY_HOME\lib\derby.jar"
Test-Path "$env:DERBY_HOME\lib\derbytools.jar"
java org.apache.derby.tools.sysinfo
```

The first command shows each classpath entry. The next commands check that key files exist and that Derby tools can start.

## Common setup problems

| Symptom | Likely cause | Check |
| --- | --- | --- |
| `java` is not recognized | JDK `bin` is missing from `PATH` | Run `java --version` in a new terminal |
| `ClassNotFoundException` for a Derby class | A required Derby jar is missing from the classpath | Print `$env:CLASSPATH` and inspect `DERBY_HOME\lib` |
| `UnsupportedClassVersionError` | Java is older than the bytecode used by the selected Derby release | Compare `java --version` with the official compatibility table |
| `ij` starts but cannot connect | The connection URL or driver mode is wrong | Check the URL format in the next chapter |
| An old database refuses to boot | Its format may need an upgrade or the runtime may be incompatible | Back up the database, then read the upgrade guidance before changing it |

## Practice

1. Record the output of `java --version`, `javac --version`, and `sysinfo`.
2. Remove `derbytools.jar` from the classpath and start `ij`. Restore the jar and explain the difference.
3. Confirm which library files are present in the distribution you downloaded.
4. Write down the Java and Derby versions used by an existing application before attempting to open its database with another installation.

## Further reading

- [Apache Derby downloads and compatibility table](https://db.apache.org/derby/derby_downloads)
- [Derby scripts and libraries](https://db.apache.org/derby/docs/10.17/getstart/rgslib27507.html)
- [Derby system requirements and deployment options](https://db.apache.org/derby/docs/10.17/getstart/cgsintro.html)

| [Previous: Relational databases and Derby](01-relational-databases-and-derby.md) | [Notes index](../README.md) | [Next: Using ij and creating a database](03-ij-and-first-database.md) |
| --- | --- | --- |
