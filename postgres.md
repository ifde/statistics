### PostgeSQL notes 

#### Order of Operations 

```sql
SELECT DISTINCT column1, AGG(column2)
FROM table1
JOIN table2 ON condition
WHERE condition
GROUP BY column1
HAVING condition
ORDER BY column1
LIMIT n;
```

```text
1. FROM / JOIN          - Load tables and perform joins
2. WHERE                - Filter rows based on conditions
3. GROUP BY             - Group rows into groups
4. HAVING               - Filter groups
5. SELECT               - Select columns and compute expressions
6. DISTINCT             - Remove duplicate rows
7. ORDER BY             - Sort results
8. LIMIT / OFFSET       - Limit number of rows returned
```

