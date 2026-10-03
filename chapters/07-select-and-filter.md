# 7. Selecting and filtering data

[Back to notes index](../README.md)

| [Previous: Inserting, updating, and deleting data](06-insert-update-delete.md) | [Notes index](../README.md) | [Next: Joins, subqueries, and views](08-joins-subqueries-and-views.md) |
| --- | --- | --- |

## Select the columns you need

`SELECT` reads rows from one or more tables. Name the columns the caller needs:

```sql
SELECT BOOK_ID, TITLE, PUBLISHED_YEAR
FROM LIBRARY.BOOKS;
```

Use an alias to make a result column easier to read:

```sql
SELECT TITLE AS BOOK_TITLE,
       PUBLISHED_YEAR AS YEAR_PUBLISHED
FROM LIBRARY.BOOKS;
```

`SELECT *` is useful while exploring a table. In application queries, explicit columns keep the result stable when a table gains a new column.

## Filter rows with WHERE

The `WHERE` clause keeps rows that match a condition. Derby supports comparisons such as `=`, `<>`, `<`, `<=`, `>`, and `>=`.

```sql
SELECT BOOK_ID, TITLE, PUBLISHED_YEAR
FROM LIBRARY.BOOKS
WHERE PUBLISHED_YEAR >= 2000;
```

Combine conditions with `AND` and `OR`. Use parentheses when the intended order should be obvious:

```sql
SELECT BOOK_ID, TITLE
FROM LIBRARY.BOOKS
WHERE AVAILABLE = TRUE
  AND (PUBLISHED_YEAR >= 2020 OR PUBLISHED_YEAR IS NULL);
```

Without parentheses, SQL evaluates `AND` before `OR`. Parentheses help readers confirm that the filter matches the intended rule.

## Handle missing values

`NULL` represents an absent or unknown value. Use `IS NULL` or `IS NOT NULL` to test it:

```sql
SELECT BOOK_ID, TITLE
FROM LIBRARY.BOOKS
WHERE PUBLISHED_YEAR IS NULL;
```

This condition does not find null values:

```sql
WHERE PUBLISHED_YEAR = NULL
```

Comparisons with `NULL` produce an unknown result. Use the dedicated null predicates instead.

## Search text and ranges

`LIKE` matches text patterns. `%` matches zero or more characters. `_` matches one character:

```sql
SELECT BOOK_ID, TITLE
FROM LIBRARY.BOOKS
WHERE TITLE LIKE '%SQL%';
```

`BETWEEN` checks an inclusive range:

```sql
SELECT TITLE, PUBLISHED_YEAR
FROM LIBRARY.BOOKS
WHERE PUBLISHED_YEAR BETWEEN 2000 AND 2025;
```

For a set of exact values, use `IN`:

```sql
SELECT BOOK_ID, TITLE
FROM LIBRARY.BOOKS
WHERE PUBLISHED_YEAR IN (1999, 2008, 2022);
```

## Sort and limit the result

Use `ORDER BY` to request a predictable order. Without it, row order is not guaranteed:

```sql
SELECT BOOK_ID, TITLE, PUBLISHED_YEAR
FROM LIBRARY.BOOKS
ORDER BY PUBLISHED_YEAR DESC, TITLE ASC;
```

Use a row limit when only a small result is needed:

```sql
SELECT BOOK_ID, TITLE
FROM LIBRARY.BOOKS
ORDER BY TITLE
FETCH FIRST 5 ROWS ONLY;
```

The `ORDER BY` is important when choosing the first rows. Without an explicit order, the selected rows can vary as data or indexes change.

## Remove duplicates and calculate values

`DISTINCT` removes duplicate result values:

```sql
SELECT DISTINCT PUBLISHED_YEAR
FROM LIBRARY.BOOKS
WHERE PUBLISHED_YEAR IS NOT NULL
ORDER BY PUBLISHED_YEAR;
```

Expressions can calculate a value for each result row:

```sql
SELECT TITLE,
       CASE
           WHEN AVAILABLE = TRUE THEN 'Available'
           ELSE 'Checked out'
       END AS AVAILABILITY_LABEL
FROM LIBRARY.BOOKS;
```

## Aggregate rows

Aggregate functions summarize rows. `COUNT`, `SUM`, `AVG`, `MIN`, and `MAX` are common examples:

```sql
SELECT COUNT(*) AS BOOK_COUNT,
       MIN(PUBLISHED_YEAR) AS OLDEST_YEAR,
       MAX(PUBLISHED_YEAR) AS NEWEST_YEAR
FROM LIBRARY.BOOKS;
```

Use `GROUP BY` to calculate one summary per group:

```sql
SELECT PUBLISHED_YEAR, COUNT(*) AS BOOK_COUNT
FROM LIBRARY.BOOKS
WHERE PUBLISHED_YEAR IS NOT NULL
GROUP BY PUBLISHED_YEAR
HAVING COUNT(*) > 1
ORDER BY PUBLISHED_YEAR;
```

`WHERE` filters input rows before grouping. `HAVING` filters the groups after aggregation.

## Common mistakes

- Expecting a stable row order without `ORDER BY`.
- Writing `= NULL` instead of `IS NULL`.
- Combining `AND` and `OR` without clarifying the intended grouping.
- Using `FETCH FIRST` without deciding which rows should come first.
- Selecting a non-aggregated column that is not part of the `GROUP BY` list.
- Using `WHERE` for an aggregate condition that belongs in `HAVING`.

## Practice

1. List available books published since 2020 and sort newest first.
2. Find titles containing the word `SQL`.
3. Count books by publication year and keep only years with more than one book.
4. Return the first three books in alphabetical order.
5. Add a `CASE` expression that labels books with a missing publication year as `Unknown`.

## Further reading

- [SELECT statement and query clauses](https://db.apache.org/derby/docs/10.17/ref/rrefsqlj41360.html)
- [WHERE clause](https://db.apache.org/derby/docs/10.17/ref/rrefsqlj33602.html)
- [Aggregates and set functions](https://db.apache.org/derby/docs/10.17/ref/rrefsqlj33923.html)

| [Previous: Inserting, updating, and deleting data](06-insert-update-delete.md) | [Notes index](../README.md) | [Next: Joins, subqueries, and views](08-joins-subqueries-and-views.md) |
| --- | --- | --- |
