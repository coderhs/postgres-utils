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

*pg_cancel_backend* - accepts only one pid at a time. It returns true or false (if its executed or not) and it you should use
cancel for an active query.

## Find rows whose attribute with timestap has time between two time of the day.

```sql
SELECT *
FROM table_name
WHERE CAST(start_time as time) NOT BETWEEN TIME '8:30:00' AND TIME '12:30:00';
```

## Find the table (within a schema) taking maximum disk space

```sql
  SELECT pg_namespace.nspname AS schema, pg_class.relname AS relation,
    pg_size_pretty(pg_total_relation_size(pg_class.oid::regclass)) AS size,
    COALESCE(pg_stat_user_tables.seq_scan + pg_stat_user_tables.idx_scan, 0) AS scans
   FROM pg_class
   LEFT JOIN pg_stat_user_tables ON pg_stat_user_tables.relid = pg_class.oid
   LEFT JOIN pg_namespace ON pg_namespace.oid = pg_class.relnamespace
  WHERE pg_class.relkind = 'r'::"char"
  AND pg_namespace.nspname NOT IN ('pg_catalog', 'information_schema')
  ORDER BY pg_total_relation_size(pg_class.oid::regclass) DESC;
```

## Create a view of all the active locks

```sql
  CREATE OR REPLACE VIEW public.active_locks AS
   SELECT t.schemaname,
      t.relname,
      l.locktype,
      l.page,
      l.virtualtransaction,
      l.pid,
      l.mode,
      l.granted
     FROM pg_locks l
     JOIN pg_stat_all_tables t ON l.relation = t.relid
    WHERE t.schemaname <> 'pg_toast'::name AND t.schemaname <> 'pg_catalog'::name
    ORDER BY t.schemaname, t.relname;
```


## Get total number of DB Connections

```sql
select count(*) from pg_stat_activity;
```

## Get total DB size

```sql
SELECT pg_size_pretty(pg_database_size('DB NAME'));
```
