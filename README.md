# Apache Derby Study Notes

These are my personal Apache Derby study notes, collected while learning database concepts and working through practical examples. They explain core SQL ideas, Derby tools, and Java JDBC access in a beginner-friendly way.

The examples focus on Apache Derby 10.17.1.0 and Java 21 or newer where a version matters. Check the official compatibility table before choosing a runtime.

> **Project status:** Apache Derby moved to retired, read-only status on October 10, 2025. The project states that development and bug fixes have ended and that no further releases will be published. These notes are useful for learning the database and maintaining existing Derby systems. Review the official status before selecting Derby for new production work.

## Core topics

1. [Relational databases and Derby architecture](chapters/01-relational-databases-and-derby.md)
2. [Java compatibility, setup, and tools](chapters/02-compatibility-setup-and-tools.md)
3. [Using ij and creating a database](chapters/03-ij-and-first-database.md)
4. [Schemas, tables, and data types](chapters/04-schemas-tables-and-data-types.md)
5. [Constraints and indexes](chapters/05-constraints-and-indexes.md)
6. [Inserting, updating, and deleting data](chapters/06-insert-update-delete.md)
7. [Selecting and filtering data](chapters/07-select-and-filter.md)
8. [Joins, subqueries, and views](chapters/08-joins-subqueries-and-views.md)
9. [Transactions and concurrency](chapters/09-transactions-and-concurrency.md)
10. [JDBC connections and prepared statements](chapters/10-jdbc-connections-and-statements.md)
11. [Embedded mode and database lifecycle](chapters/11-embedded-mode-and-lifecycle.md)
12. [Network Server and client connections](chapters/12-network-server-and-client.md)
13. [Metadata and schema changes](chapters/13-metadata-and-schema-changes.md)
14. [Import, export, backup, and recovery](chapters/14-import-export-backup-and-recovery.md)
15. [Authentication, authorization, and security](chapters/15-authentication-authorization-and-security.md)
16. [Testing and troubleshooting](chapters/16-testing-and-troubleshooting.md)
17. [Performance and operating considerations](chapters/17-performance-and-operations.md)
18. [All code samples](chapters/98-all-code-samples.md)
19. [Complete questions and answers](chapters/99-complete-q-and-a.md)

## How to use these notes

Read the chapters in order. The SQL examples build a small library database, then the Java examples connect to it with JDBC. Each topic explains the purpose of the commands, shows working examples, and calls out common mistakes. The final chapters collect the code and review questions for quick reference.

## Official references

- [Apache Derby manuals](https://db.apache.org/derby/manuals/)
- [Apache Derby downloads and Java compatibility](https://db.apache.org/derby/derby_downloads)
- [Apache Derby FAQ](https://db.apache.org/derby/faq.html)
- [Apache Derby 10.17 Reference Manual](https://db.apache.org/derby/docs/10.17/ref/)
- [Apache Derby 10.17 Developer's Guide](https://db.apache.org/derby/docs/10.17/devguide/)
- [Apache Derby 10.17 Server and Administration Guide](https://db.apache.org/derby/docs/10.17/adminguide/)

## Links

- Portfolio: [ashishranjan.net](https://www.ashishranjan.net)
- GitHub: [github.com/a2rp](https://github.com/a2rp)
- CodePen: [codepen.io/ash1198](https://codepen.io/ash1198)
- LinkedIn: [linkedin.com/in/aashishranjan](https://www.linkedin.com/in/aashishranjan)
- Facebook: [facebook.com/theash.ashish](https://www.facebook.com/theash.ashish/)
- YouTube: [Ashish Ranjan](https://www.youtube.com/@ashishranjan-ashz?sub_confirmation=1)
- Email: [ash.ranjan09@gmail.com](mailto:ash.ranjan09@gmail.com)

## Support

- [Support](https://a2rp-donation-page.netlify.app/)
- [Buy Me a Coffee](https://buymeacoffee.com/ashishranjan)
- [Patreon](https://www.patreon.com/ashishranjan)
