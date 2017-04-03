## Display query which are in lock, how long they have been running

```sql
SELECT a.datname,
         c.relname,
         l.transactionid,
         l.mode,
         l.GRANTED,
         a.usename,
         a.query, 
         a.query_start,
         age(now(), a.query_start) AS "age", 
         a.pid 
    FROM  pg_stat_activity a
     JOIN pg_locks         l ON l.pid = a.pid
     JOIN pg_class         c ON c.oid = l.relation
    ORDER BY a.query_start;
```

## Terminate a runing query

```sql
SELECT pid, age(now(), query_start) AS "age", pg_cancel_backend(pid)
FROM pg_stat_activity
WHERE query != '<IDLE>' AND query NOT ILIKE '%pg_stat_activity%' AND query ILIKE '%DELETE%';
```

pg_cancel_backend - accepts only one pid at a time. It returns true or false (if its executed or not) and it you should use
cancel for an active query
