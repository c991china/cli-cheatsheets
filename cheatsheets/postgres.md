# postgres cheatsheet

psql 15/16. `\d` commands are psql meta-commands; the rest is SQL.

## Connecting

```bash
psql -h localhost -U postgres -d mydb
psql "postgresql://user:pass@host:5432/mydb"
psql -h host -U user -d db -c 'SELECT 1'
PGPASSWORD=secret psql -h host -U user db     # password without prompt
psql -f schema.sql -d mydb                     # run a file
\c mydb                                        # switch database
\conninfo                                      # current connection
```

## psql meta-commands

```
\l              list databases
\c db           connect to db
\dt             list tables
\dt+            + sizes
\d table        describe table (columns, indexes)
\d+ table       more detail
\di             list indexes
\di+            + sizes
\df             list functions
\dn             list schemas
\du             list roles
\dv             list views
\dp             list permissions
\x              toggle expanded output (wide rows)
\e              edit the current query in $EDITOR
\i file.sql     run a file
\o out.txt      send query output to a file
\timing         show query time
\watch 5        re-run last query every 5s
\copy t FROM 'x.csv' CSV HEADER
\q              quit
```

`\x` is the one I use most. Wide rows become readable instantly.

## Reading / searching

```sql
SELECT * FROM users LIMIT 10;
SELECT count(*) FROM users;
SELECT column_name, data_type FROM information_schema.columns
WHERE table_name = 'users';
SELECT tablename FROM pg_tables WHERE schemaname = 'public';
SELECT * FROM pg_stat_activity;             -- current queries
SHOW search_path;
```

## Common DDL

```sql
CREATE TABLE users (
  id         bigserial PRIMARY KEY,
  email      text NOT NULL UNIQUE,
  created_at timestamptz NOT NULL DEFAULT now()
);

ALTER TABLE users ADD COLUMN name text;
ALTER TABLE users ALTER COLUMN email SET NOT NULL;
ALTER TABLE users DROP COLUMN name;
DROP TABLE IF EXISTS users CASCADE;

CREATE INDEX CONCURRENTLY idx_users_email ON users (email);
CREATE INDEX idx_users_created ON users (created_at DESC);
DROP INDEX CONCURRENTLY idx_users_email;
```

Use `timestamptz`, not `timestamp`. And `CONCURRENTLY` for indexes on live tables.

## Upserts / returning

```sql
INSERT INTO users (email) VALUES ('a@b.com')
ON CONFLICT (email) DO UPDATE SET email = EXCLUDED.email
RETURNING id;

INSERT INTO users (email) VALUES ('x@y.com')
ON CONFLICT (email) DO NOTHING;

UPDATE users SET name = 'ann' WHERE id = 1 RETURNING *;
DELETE FROM users WHERE created_at < now() - interval '1 year';
```

`RETURNING` gives you the affected rows back without a second SELECT.

## JSON

```sql
SELECT data->>'name' FROM t;                 -- text
SELECT data->'tags' FROM t;                  -- json
SELECT data#>>'{a,b}' FROM t;                -- nested path as text
SELECT * FROM t WHERE data->>'status' = 'open';
SELECT * FROM t WHERE data @> '{"role":"admin"}';
SELECT jsonb_array_elements(data->'items') FROM t;   -- expand array
CREATE INDEX ON t USING gin (data jsonb_path_ops);
```

`->` keeps json, `->>` returns text. `#>>` for nested paths.

## Transactions / locks

```sql
BEGIN;
UPDATE accounts SET bal = bal - 100 WHERE id = 1;
UPDATE accounts SET bal = bal + 100 WHERE id = 2;
COMMIT;                -- or ROLLBACK;

SELECT pg_blocking_pids(<pid>);
SELECT * FROM pg_locks WHERE NOT granted;
```

## Performance / ops

```sql
EXPLAIN ANALYZE SELECT ...;
VACUUM (VERBOSE, ANALYZE) t;
VACUUM FULL t;                          -- locks table for writes
ANALYZE t;
REINDEX INDEX CONCURRENTLY idx;
SELECT pg_size_pretty(pg_total_relation_size('t'));
SET statement_timeout = '5s';
SELECT pg_terminate_backend(<pid>);     -- kill a session
SELECT pg_cancel_backend(<pid>);        -- cancel a query, keep the session
```

## Backup / restore

```bash
pg_dump mydb > mydb.sql
pg_dump -Fc mydb > mydb.dump           # custom format, compressed
pg_dump -t users mydb > users.sql      # one table
pg_dump -s mydb > schema.sql           # schema only
psql mydb < mydb.sql
pg_restore -d mydb mydb.dump
pg_restore -d mydb -j 4 mydb.dump      # parallel restore
```

`-Fc` + `pg_restore` is faster and lets you restore selectively. Plain `.sql` is
fine for small databases.

## Notes

- `count(*)` on a big table is a full scan under MVCC. It's slow by design.
- Type `\x` before running a wide query. It saves a lot of squinting.
- `pg_terminate_backend` on the wrong pid kills your own session. Check twice.
- `CREATE INDEX` (without CONCURRENTLY) takes a write lock. On a busy table,
  that's an outage.
