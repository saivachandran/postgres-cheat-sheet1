# Useful Postgres Commands

### 1. Determine disk usage of a particular table/database

For a particular table,

```
SELECT pg_size_pretty( pg_total_relation_size('tablename') );
```

For a particular database,

```
SELECT pg_size_pretty( pg_database_size('dbname') );
```

### 2. Show config file location

```
$ psql -U postgres -c 'SHOW config_file'
```

### 3. Export data from a database table to a CSV file

```
COPY (SELECT "dbname".* FROM "tablename")
  TO '/tmp/output-file.csv'
  WITH DELIMITER ';' CSV HEADER
```

### 4. Import data from a CSV file to database

```
COPY tablename
  FROM '/tmp/input-file.csv'
```

### 5. Grant privileges

```
REVOKE ALL ON DATABASE example_database FROM example_user;
GRANT CONNECT ON DATABASE example_database TO example_user;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO example_user;
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT ON TABLES TO example_user;

# Grant all privileges on all tables to a particular user
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO example_user;

# Grant all privileges on all tables (both existing and new tables) to a certain user
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT ALL PRIVILEGES ON TABLES TO example_user;

# View a user's grants
SELECT table_catalog, table_schema, table_name, privilege_type FROM information_schema.table_privileges WHERE grantee = 'example_user' ORDER BY table_name ASC;
```

OR

```
GRANT USAGE ON SCHEMA public TO my_db_user;
GRANT SELECT, UPDATE, INSERT, DELETE ON ALL TABLES IN SCHEMA public TO my_db_user;

-- If you have sequences
GRANT SELECT, UPDATE, USAGE ON ALL SEQUENCES IN SCHEMA public to my_db_user;

-- If you have functions
GRANT EXECUTE ON ALL FUNCTIONS IN SCHEMA public TO my_db_user;


-- Cater for future tables that will be created
ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT, UPDATE, INSERT, DELETE, TRIGGER ON TABLES TO my_db_user;
```

### 6. Find slow queries

Check that you have the pg_stat_statements extension installed

```
 postgres=# \x
 postgres=# \dx pg_stat_statements
```

If you dnn't obtain any result, then issue the following command:

```
postgres=# CREATE EXTENSION pg_stat_statements; 
postgres=# ALTER SYSTEM
          SET shared_preload_libraries = 'pg_stat_statements';
```

Then, restart the server

You can get the top ten highest workloads on your server side by executing the following:

```
postgres=# SELECT calls, total_time, query FROM pg_stat_statements 
           ORDER BY total_time DESC LIMIT 10;
```

There are many additional columns that are useful in tracking down further information about particular entries

```
postgres=# \d pg_stat_statements 
          View "public.pg_stat_statements" 
       Column        |       Type       | Modifiers  
---------------------+------------------+----------- 
 userid              | oid              |  
 dbid                | oid              |  
 queryid             | bigint           |  
 query               | text             |  
 calls               | bigint           |  
 total_time          | double precision |  
 min_time            | double precision |  
 max_time            | double precision |  
 mean_time           | double precision |  
 stddev_time         | double precision |  
 rows                | bigint           |  
 shared_blks_hit     | bigint           |  
 shared_blks_read    | bigint           |  
 shared_blks_dirtied | bigint           |  
 shared_blks_written | bigint           |  
 local_blks_hit      | bigint           |  
 local_blks_read     | bigint           |  
 local_blks_dirtied  | bigint           |  
 local_blks_written  | bigint           |  
 temp_blks_read      | bigint           |  
 temp_blks_written   | bigint           |  
 blk_read_time       | double precision |  
 blk_write_time      | double precision |
```

### 7. Find slow queries that takes over 10 seconds

```
postgres=# ALTER SYSTEM
          SET log_min_duration_statement = 10000;
```

### 8. Rename table

```
ALTER TABLE IF EXISTS table_name
RENAME TO new_table_name;
```

### 9. Get database owner

```
SELECT d.datname as "Name",
pg_catalog.pg_get_userbyid(d.datdba) as "Owner"
FROM pg_catalog.pg_database d
WHERE d.datname = 'db_name'
ORDER BY 1;
```

or if you are using the `psql`:

```
# \l db_name
```

### 10. Bulk load CSV file into postgres table

Assuming the table `my_table` already exists,

```
\copy my_table (col1, col2, col3, ...) FROM '/path/to/csv-file.csv' CSV HEADER;
```

### 11. Make a dump (copy) of your database

```
$ pg_dump <dbname> -U <dbuser> -f <filename>.sql

# If you want to compress the database dump
$ pg_dump <dbname> -U <dbuser> | gzip > <filename>.sql.gz
```

### 12. To restore database dump copy on another system

1. Create an empty database or clear out the existing database

```
# To clear out an existing database/schema
$ psql -U <dbuser> -d <dbname> -c "DROP SCHEMA IF EXISTS public CASCADE; CREATE SCHEMA public;"
```

2. Gunzip the copy, if you created a compressed version
3. Run the following command to restore:

```
$ psql -d <dbname> -U <dbuser> -f <filename>.sql
```


### 13. Select random records

```
SELECT *
FROM words
WHERE difficulty = 'Easy' AND category_id = 3
ORDER BY random()
LIMIT 1;
```

### 14. List all the tables in a particular schema

```
SELECT * FROM information_schema.tables WHERE table_schema = 'public';
```

Alternatively,

For all schemas

```
=> \dt *.*
```

For a particular schema,

```
=> \dt public.*
```

### 15. Locate Postgres configuration files

```
SHOW hba_file;

SHOW config_file;
```


### 16. How to reload config settings without restarting database

```
su - postgres

/usr/bin/pg_ctl reload
```

Alternatively, you can use SQL

```
SELECT pg_reload_conf();
```


### 17. Select bytea column data as string

```
SELECT convert_from(decode(<bytea_column>, 'escape'), 'UTF-8') FROM <table_name>;
```


### 18. How to determine the PostgreSQL and PostGIS versions

From the command line,

```shell script
psql --version
```

Alternatively, from _PSQL_ program

```sql
SELECT version(); # postgres version

SELECT PostGIS_full_version(); # postgis version
```

### 19. [🔗](https://www.postgresql.org/docs/current/sql-createuser.html) Create PostgreSQL User

To create a postgreSQL user with a create database permission only, you can use any of the approaches below:

```sql
# Without password
CREATE ROLE myuser;
ALTER ROLE myuser WITH CREATEDB;

# with password
CREATE ROLE myuser;
ALTER ROLE myuser WITH LOGIN ENCRYPTED PASSWORD 'somepassword' CREATEDB;
```

Alternatively,

```sql
# Without password
CREATE ROLE myuser CREATEDB;

# With password (Option 1)
CREATE ROLE myuser WITH LOGIN ENCRYPTED PASSWORD 'somepassword' CREATEDB;

# With password and superuser privileges (Option 2)
CREATE USER myuser WITH SUPERUSER CREATEDB LOGIN ENCRYPTED PASSWORD 'somepassword';
```

> If `CREATEDB` is specified, the created user will be allowed to create their own databases. 
> Using `NOCREATEDB` will deny the user the ability to create databases. If not specified, `NOCREATEDB` is the default

### 20. Change PostgreSQL User Password

To change the password of a Postgres user:

1. Login to Postgres without a password

```
sudo -u <user_name> psql db_name
```

For example,

```
sudo -u postgres psql dhis2
```

2. Reset the password

```
ALTER USER <user_name> WITH PASSWORD '<new_password>';
```

Alternatively,

```
sudo -u postgres psql

\password postgres
```

### 21. Hide result set decoration in Psql output

To hide the column names included as part of the query resultset in psql, you can pass the `-t` or `--tuples-only` flag to psql:

```
$ psql --user=dbuser -d mydb -t -c "SELECT count(*) FROM dbtable;"
```

### 22. Check if a Postgres database exist (case-insensitive way)

```
SELECT datname FROM pg_catalog.pg_database WHERE lower(datname) = 'db-name-in-lower-case';
```

Alternatively from the command line,

```
psql -U postgres -tc "SELECT 1 FROM pg_database WHERE datname = 'db-name'" | grep -q 1 || psql -U postgres -c "CREATE DATABASE db-name"
```

### 23. Check if a Postgres user exist

```
SELECT 1 FROM pg_roles WHERE rolname='the-postgres-username'
```

If you'd rather run it from the command line:

```
psql postgres -tAc "SELECT 1 FROM pg_roles WHERE rolname='the-postgres-username'"
```

On unix, you can use grep to chain multiple commands:

```
psql postgres -tAc "SELECT 1 FROM pg_roles WHERE rolname='the-postgres-username'" | grep -q 1 || createuser ...
```

### 24. Check if a Postgres schema exist

```
SELECT schema_name FROM information_schema.schemata WHERE schema_name = 'schema-name';
```

If you will like to create a schema if one doesn't already exist,

```
CREATE SCHEMA IF NOT EXISTS `schema-name`;
```

Likewise for Postgres extensions:

```
CREATE EXTENSION IF NOT EXISTS `extension-name`;
```

### 25. Query active Postgres configuration parameter value

```sql
SELECT name, setting FROM pg_settings;
SELECT setting FROM pg_settings WHERE name = 'max_locks_per_transaction';
```

### 26. Check if a table column exists

```
SELECT column_name FROM information_schema.columns WHERE table_name='the_table_name' and column_name='the_column_name';

# OR

SELECT column_name FROM information_schema.columns WHERE table_name='the_table_name' and column_name='the_column_name';
```

Alternatively, you can adapt the query to return `true/false`:

```sql
SELECT EXISTS (SELECT 1 FROM information_schema.columns WHERE table_schema='my_schema_name' AND table_name='my_table_name' AND column_name='my_column_name');
```

### 27. List installed postgresql extensions

```
postgres=# \dx
                 List of installed extensions
  Name   | Version |   Schema   |         Description          
---------+---------+------------+------------------------------
 plpgsql | 1.0     | pg_catalog | PL/pgSQL procedural language
 postgis | 2.4.4   | postgis    | PostGIS geometry, geography, and raster spatial types and functions
(2 rows)
```

To get more details,

```
postgres=# \dx+ plpgsql
      Objects in extension "plpgsql"
            Object Description             
-------------------------------------------
 function plpgsql_call_handler()
 function plpgsql_inline_handler(internal)
 function plpgsql_validator(oid)
 language plpgsql
(4 rows)
```

### 28. List all available schemas

1. Using SQL Query

You can get the list of all schemas using SQL with the ANSI standard of `INFORMATION_SCHEMA`:

```
SELECT schema_name FROM information_schema.schemata
```

or

```
SELECT nspname FROM pg_catalog.pg_namespace;
```

Information schema is simply a set of views of `pg_catalog`.

2. Using psql

```
\dn
```

### 29. [🔗](https://www.postgresql.org/docs/current/sql-droprole.html) Drop User Role If Exists

```
DROP ROLE IF EXISTS bambini;
```
## Get table size
```
select schemaname as table_schema,
    relname as table_name,
    pg_size_pretty(pg_total_relation_size(relid)) as total_size,
    pg_size_pretty(pg_relation_size(relid)) as data_size,
    pg_size_pretty(pg_total_relation_size(relid) - pg_relation_size(relid))
      as external_size
from pg_catalog.pg_statio_user_tables
order by pg_total_relation_size(relid) desc,
         pg_relation_size(relid) desc
limit 10;
```
# get dead tuple
```
SELECT schemaname, relname, n_live_tup, n_dead_tup, last_autovacuum
FROM pg_stat_all_tables
ORDER BY n_dead_tup
    / (n_live_tup
       * current_setting('autovacuum_vacuum_scale_factor')::float8
          + current_setting('autovacuum_vacuum_threshold')::float8)
     DESC
LIMIT 20;
```
