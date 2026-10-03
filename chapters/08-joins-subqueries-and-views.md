# 8. Joins, subqueries, and views

[Back to notes index](../README.md)

| [Previous: Selecting and filtering data](07-select-and-filter.md) | [Notes index](../README.md) | [Next: Transactions and concurrency](09-transactions-and-concurrency.md) |
| --- | --- | --- |

## Combine related rows with a join

A join reads related rows from more than one table. In the library database, `LOANS.BOOK_ID` refers to `BOOKS.BOOK_ID`, while `LOANS.MEMBER_ID` refers to `MEMBERS.MEMBER_ID`.

The previous chapter defines `LIBRARY.BOOKS`, `LIBRARY.MEMBERS`, and `LIBRARY.LOANS`. In a fresh practice database, continue from that schema and add a second member plus two loan rows. Run this setup once:

```sql
INSERT INTO LIBRARY.MEMBERS (MEMBER_ID, FULL_NAME, EMAIL)
VALUES (2, 'Dev Shah', 'dev@example.com');

INSERT INTO LIBRARY.LOANS
    (BOOK_ID, MEMBER_ID, CHECKED_OUT_ON, RETURNED_ON)
VALUES (1, 1, DATE '2026-09-01', NULL),
       (2, 2, DATE '2026-09-03', DATE '2026-09-10');
```

An inner join returns rows where the join condition matches on both sides:

```sql
SELECT b.BOOK_ID,
       b.TITLE,
       m.FULL_NAME,
       l.CHECKED_OUT_ON,
       l.RETURNED_ON
FROM LIBRARY.LOANS AS l
INNER JOIN LIBRARY.BOOKS AS b
    ON b.BOOK_ID = l.BOOK_ID
INNER JOIN LIBRARY.MEMBERS AS m
    ON m.MEMBER_ID = l.MEMBER_ID
ORDER BY l.CHECKED_OUT_ON;
```

The aliases `b`, `l`, and `m` make longer table names easier to read. Prefix each selected column with its alias when a name appears in more than one table, such as `BOOK_ID`.

## Keep unmatched rows with a left join

A left outer join returns every row from the table on the left. When no row matches on the right, Derby returns `NULL` for the right-side columns:

```sql
SELECT b.BOOK_ID,
       b.TITLE,
       l.CHECKED_OUT_ON,
       m.FULL_NAME
FROM LIBRARY.BOOKS AS b
LEFT OUTER JOIN LIBRARY.LOANS AS l
    ON l.BOOK_ID = b.BOOK_ID
   AND l.RETURNED_ON IS NULL
LEFT OUTER JOIN LIBRARY.MEMBERS AS m
    ON m.MEMBER_ID = l.MEMBER_ID
ORDER BY b.BOOK_ID;
```

The condition on `RETURNED_ON` is part of the `ON` clause. This keeps books without an active loan in the result. Moving that condition to `WHERE` would remove rows where the loan columns are `NULL`, changing the result into an effective inner filter.

## Find rows with a subquery

A subquery is a query nested inside another SQL statement. Use `IN` when the inner query returns a list of values:

```sql
SELECT b.BOOK_ID, b.TITLE
FROM LIBRARY.BOOKS AS b
WHERE b.BOOK_ID IN (
    SELECT l.BOOK_ID
    FROM LIBRARY.LOANS AS l
)
ORDER BY b.BOOK_ID;
```

Use `EXISTS` when the question is whether at least one related row exists. This correlated subquery refers to the current member through `m.MEMBER_ID`:

```sql
SELECT m.MEMBER_ID, m.FULL_NAME
FROM LIBRARY.MEMBERS AS m
WHERE EXISTS (
    SELECT 1
    FROM LIBRARY.LOANS AS l
    WHERE l.MEMBER_ID = m.MEMBER_ID
      AND l.RETURNED_ON IS NULL
)
ORDER BY m.MEMBER_ID;
```

Use `NOT EXISTS` to find members with no matching loan rows:

```sql
SELECT m.MEMBER_ID, m.FULL_NAME
FROM LIBRARY.MEMBERS AS m
WHERE NOT EXISTS (
    SELECT 1
    FROM LIBRARY.LOANS AS l
    WHERE l.MEMBER_ID = m.MEMBER_ID
)
ORDER BY m.MEMBER_ID;
```

`NOT EXISTS` is often easier to reason about than `NOT IN` when the subquery could return `NULL` values.

## Use a scalar subquery

A scalar subquery returns one row with one column. An aggregate such as `COUNT(*)` returns one value, so it can be selected beside each book:

```sql
SELECT b.BOOK_ID,
       b.TITLE,
       (
           SELECT COUNT(*)
           FROM LIBRARY.LOANS AS l
           WHERE l.BOOK_ID = b.BOOK_ID
       ) AS LOAN_COUNT
FROM LIBRARY.BOOKS AS b
ORDER BY b.BOOK_ID;
```

The inner query uses the current book row. This is a correlated scalar subquery. A scalar subquery that returns more than one row causes an error, so use an aggregate or a condition that guarantees at most one result.

## Use a query as a temporary table

A subquery in the `FROM` clause can calculate a result that the outer query joins like a table. Give the derived table a name:

```sql
SELECT b.BOOK_ID,
       b.TITLE,
       loan_counts.LOAN_COUNT
FROM LIBRARY.BOOKS AS b
LEFT JOIN (
    SELECT l.BOOK_ID, COUNT(*) AS LOAN_COUNT
    FROM LIBRARY.LOANS AS l
    GROUP BY l.BOOK_ID
) AS loan_counts
    ON loan_counts.BOOK_ID = b.BOOK_ID
ORDER BY b.BOOK_ID;
```

This query summarizes loan rows once in the derived table, then joins those totals to books. Books without loans have no matching summary row, so their `LOAN_COUNT` is `NULL`. Use `COALESCE(loan_counts.LOAN_COUNT, 0)` if the result should display zero instead.

## Save a useful query as a view

A view gives a query a reusable database name. It reads from its underlying tables when selected:

```sql
CREATE VIEW LIBRARY.ACTIVE_LOANS
    (BOOK_TITLE, MEMBER_NAME, CHECKED_OUT_ON)
AS
    SELECT b.TITLE, m.FULL_NAME, l.CHECKED_OUT_ON
    FROM LIBRARY.LOANS AS l
    INNER JOIN LIBRARY.BOOKS AS b
        ON b.BOOK_ID = l.BOOK_ID
    INNER JOIN LIBRARY.MEMBERS AS m
        ON m.MEMBER_ID = l.MEMBER_ID
    WHERE l.RETURNED_ON IS NULL;
```

Query the view like a table:

```sql
SELECT BOOK_TITLE, MEMBER_NAME, CHECKED_OUT_ON
FROM LIBRARY.ACTIVE_LOANS
ORDER BY CHECKED_OUT_ON;
```

In Derby, views are not updatable. Use the base tables for `INSERT`, `UPDATE`, and `DELETE`. Remove a view with `DROP VIEW LIBRARY.ACTIVE_LOANS` when it is no longer needed.

## Choose the right approach

| Need | Common choice |
| --- | --- |
| Return rows with a match in each table | `INNER JOIN` |
| Keep every left-side row, even without a match | `LEFT OUTER JOIN` |
| Test whether related rows exist | `EXISTS` |
| Compare a value with results from another query | `IN` or a scalar subquery |
| Reuse a named read query | View |

## Common mistakes

- Joining columns with similar names but no real relationship.
- Leaving a join column unqualified when two tables contain that name.
- Putting a right-table filter in `WHERE` when unmatched left rows must remain.
- Assuming a scalar subquery may return several rows.
- Using `NOT IN` without considering `NULL` values in its result.
- Expecting a Derby view to accept data changes.
- Forgetting that one parent row can match several child rows and therefore appear more than once.

## Practice

1. Add one more member and one loan, then list each member's loaned book titles.
2. List every book, including books with no loan history.
3. Find members who have never borrowed a book using `NOT EXISTS`.
4. Return a loan count for every book using a scalar subquery, then using a derived table.
5. Create a view of all loan history, including return dates, and query it in date order.
6. Add an additional loan for the same book and observe why a join can return that book on multiple rows.

## Further reading

- [INNER JOIN operation](https://db.apache.org/derby/docs/10.17/ref/rrefsqlj35034.html)
- [LEFT OUTER JOIN operation](https://db.apache.org/derby/docs/10.17/ref/rrefsqlj18922.html)
- [Scalar subquery](https://db.apache.org/derby/docs/10.17/ref/rrefscalarsubquery.html)
- [Table subquery](https://db.apache.org/derby/docs/10.17/ref/rreftablesubquery.html)
- [CREATE VIEW statement](https://db.apache.org/derby/docs/10.17/ref/rrefsqlj15446.html)

| [Previous: Selecting and filtering data](07-select-and-filter.md) | [Notes index](../README.md) | [Next: Transactions and concurrency](09-transactions-and-concurrency.md) |
| --- | --- | --- |
