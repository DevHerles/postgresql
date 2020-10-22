# PostgreSQL for Full-Stack developers

Most useful commands for `PostgreSQL`

#### Note: `❯` is my console prompt, usually is `$`

## 1. Server configuration

### 1.1. Allow Remote Connections

Open `postgresql` configuration file located at:

```bash
❯ sudo vim /etc/postgresql/12/main/postgresql.conf
```

And change:

```bash
listen_addresses = 'localhost'
```

To:

```bash
listen_addresses = '*'
```

Then edit the `postgresql` host based authentication (hba) file located at:

```bash
❯ sudo vim /etc/postgresql/12/main/pg_hba.conf
```

If you want to allow `postgres` trusted connections, change `peer` to `trust`

```bash
# Database administrative login by Unix domain socket
local   all             postgres         0.0.0.0/0                       trust
```

<!-- https://www.postgresql.org/docs/9.1/auth-pg-hba-conf.html -->

And add the following lines in `# IPv4 local connections:` section.

```bash
host    all             all              0.0.0.0/0                       md5
host    all             all              ::/0                            md5
```

Restart `postgresql` service:

```bash
❯ sudo systemctl restart postgresql
```

<!-- https://stackoverflow.com/questions/17633422/psql-fatal-database-user-does-not-exist -->

## 2. Connect to `PostgreSQL` server

<!-- FATAL: Peer authentication failed for user (postgres) -->
<!-- https://medium.com/@wajeeh.ahsan/fatal-peer-authentication-failed-for-user-postgres-954e061c7368 -->

### 2.1. Trusted connection

If trusted connection is enabled in `pg_hba.conf` (check previous section)

```bash
❯ psql -U postgres
psql (12.4 (Ubuntu 12.4-0ubuntu0.20.04.1))
Type "help" for help.

postgres=#
```

### 2.2. Common way

First, switch to the `PostgreSQL` user, after that enter to `psql`.

```bash
❯ sudo su - postgres
[sudo] password for herles:
postgres@DELL:~$
postgres@DELL:~$ psql
psql (12.4 (Ubuntu 12.4-0ubuntu0.20.04.1))
Type "help" for help.

postgres=#
```

#### 2.2.1. Change `postgres` password

```bash
postgres=# \password postgres
Enter new password:
Enter it again:
postgres=#
```

### 2.3. To specific database `DOCKER_DB_SAMU` with host `hocalhost`, port `5432` and user `odoo`

```bash
❯ psql -h localhost -p 5432 -U odoo DOCKER_DB_SAMU
Password for user odoo:
psql (12.4 (Ubuntu 12.4-0ubuntu0.20.04.1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, bits: 256, compression: off)
Type "help" for help.

DOCKER_DB_SAMU=#
```

### 2.4. Or simply

```bash
❯ psql
psql: FATAL: database "herles" does not exist
❯
```

In this particular case, you have to create the database

```bash
❯ createdb herles
❯
```

Note:

```bash
❯ createdb herles
createdb: error: could not connect to database template1: FATAL: role "herles" does not exist
❯
```

Then, enter with the first way and create missing role. Don't forget to add `WITH LOGIN`

And try again

```bash
❯ psql
psql (12.4 (Ubuntu 12.4-0ubuntu0.20.04.1))
Type "help" for help.

herles=#
```

## 3. Manage roles

### 3.1. Create a `role` as `SUPERUSER`

```bash
postgres=# CREATE ROLE odoo WITH SUPERUSER CREATEDB CREATEROLE LOGIN REPLICATION INHERIT ENCRYPTED PASSWORD 'odoo';
CREATE ROLE
postgres=#
```

### 3.2 Alter a `role`

```bash
postgres=# ALTER ROLE odoo WITH LOGIN;
ALTER ROLE
postgres=#
```

### 3.3. Drop a `role`

```bash
postgres=# DROP ROLE odoo
postgres=#
```

#### Note:

ERROR: role `"odoo"` cannot be dropped because some objects depend on it
DETAIL: owner of database `DOCKER_DB_SAMU` (see 12 item)

In this case, the reliable sequence of commands to drop a role is:

```bash
postgres=# REASSIGN OWNED BY odoo TO postgres; -- or some other trusted role
postgres=# DROP OWNED BY odoo; -- repeat both in ALL databases where the role owns anything or has any privileges!
postgres=# DROP USER odoo;
```

<!-- https://www.postgresql.org/docs/8.1/role-membership.html -->
<!-- https://www.postgresqltutorial.com/postgresql-alter-database/ -->

### 3.4. List existing roles

```psql
postgres=# SELECT rolname FROM pg_roles;
```

```bash
          rolname
---------------------------
 herles
 postgres
 pg_monitor
 pg_read_all_settings
 pg_read_all_stats
 pg_stat_scan_tables
 pg_read_server_files
 pg_write_server_files
 pg_execute_server_program
 pg_signal_backend
 odoo
(11 rows)

postgres=#
```

Or just try: `\dg`:

```bash
postgres=# \dg
                                  List of roles
 Role name |                         Attributes                         | Member of
-----------+------------------------------------------------------------+-----------
 odoo      | Create DB                                                  | {}
 herles    | Superuser, Create role, Create DB                          | {}
 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
```

Or, even you can get more info with: `\dgS`:

```bash
postgres=# \dgS
                                                                     List of roles
         Role name         |                         Attributes                         |                          Member of
---------------------------+------------------------------------------------------------+--------------------------------------------------------------
 odoo                      | Create DB                                                  | {}
 herles                    | Superuser, Create role, Create DB                          | {}
 pg_execute_server_program | Cannot login                                               | {}
 pg_monitor                | Cannot login                                               | {pg_read_all_settings,pg_read_all_stats,pg_stat_scan_tables}
 pg_read_all_settings      | Cannot login                                               | {}
 pg_read_all_stats         | Cannot login                                               | {}
 pg_read_server_files      | Cannot login                                               | {}
 pg_signal_backend         | Cannot login                                               | {}
 pg_stat_scan_tables       | Cannot login                                               | {}
 pg_write_server_files     | Cannot login                                               | {}
 postgres                  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}

postgres=#
```

## 4. Manage `database`

### 4.1. Create `database`

With previous created role `odoo`

```bash
$ sudo su - postgres
postgres@HP:~$ createdb DOCKER_DB_ODOOPROD --owner=odoo
```

Or whit in `psql` also with role `odoo`

```bash
postgresql=# CREATE DATABASE DOCKER_DB_ODOOPROD OWNER odoo;
```

Without role (by default owner is `postgres`)

```bash
$ sudo su - postgres
postgres@HP:~$ createdb DOCKER_DB_ODOOPROD
```

Or whit in `psql`, without role

```bash
postgresql=# CREATE DATABASE DOCKER_DB_ODOOPROD;
```

### 4.2. Change the owner of the `database`

```bash
postgres=# ALTER TABLE DOCKER_DB_ODOOPROD OWNER TO new_owner | current_user | session_user;
```

### 4.3. Drop a `database`

```bash
$ sudo su - postgres
postgres@HP:~$ dropdb DOCKER_DB_ODOOPROD
```

### 4.4. Switch to another `database`

Use `\c DATABASE_NAME`

```bash
DOCKER_DB_SAMU=# \c DOCKER_DB_ODOOPROD
You are now connected to database "DOCKER_DB_ODOOPROD" as user "postgres".
DOCKER_DB_ODOOPROD=#
```

### 4.5. Backup `database`

Note: This commands are not executed whit in `psql`

#### 4.5.1. Specific `database`

You can dump the content of a database to a file like `.bak, .sql, .tar` by running the `pg_dump` command.

Remember, `DOCKER_DB_SAMU` is the database.

```bash
❯ sudo su - postgres
[sudo] password for herles:
postgres@HP:~$ pg_dump DOCKER_DB_SAMU > docker_db_samu.bak
postgres@HP:~$ ls *.bak
docker_db_samu.bak
postgres@HP:~$
```

#### 4.5.2. All `databases`

To backup all your databases at the same time, use `pg_dumpall` command.

```bash
❯ sudo su - postgres
[sudo] password for herles:
postgres@HP:~$ pg_dumpall > samu_server_db_backup.bak
postgres@HP:~$ ls *.bak
samu_server_db_backup.bak
postgres@HP:~$
```

### 4.6. Restore `database`

Note: This commands are not executed whit in `psql`

#### 4.6.1. Restore a specific `database`

To demonstrate restoring the previous `docker_db_samu.bak` backup file, we are going to create a new database `DOCKER_DB_SAMU_BAK` and use the `psql` command

```bash
❯ sudo su - postgres
[sudo] password for herles:
postgres@HP:~$ createdb DOCKER_DB_SAMU_BAK
postgres@HP:~$ psql DOCKER_DB_SAMU_BAK < docker_db_samu.bak
postgres@HP:~$
```

#### 4.6.2. Restore all `databases`

```bash
❯ sudo su - postgres
[sudo] password for herles:
postgres@HP:~$ psql -f samu_server_db_backup.bak postgres
postgres@HP:~$
```

#### 4.6.3 Restore from `psql` client

```bash
❯ psql -U postgres --file docker_db_samu.bak
```

## 5. Manage tables

### 5.1. Create a `table`

#### 5.1.1. Common way

```sql
postgres=# CREATE TABLE new_table (code varchar UNIQUE NOT NULL, name varchar);
```

#### 5.1.1. By query result

```sql
postgres=# CREATE TABLE other_table AS (SELECT code, name FROM new_table);
```

### 5.2. Alter a `table` - ADD COLUMN

```sql
postgres=# ALTER TABLE new_table ADD COLUMN code varchar;
```

### 5.3. Import data in a given table

```sql
MY_DATABASE=# COPY new_table(id, code) FROM '/path/data.csv' DELIMITER ',' CSV HEADER;
```

### 5.4. Export data from a given table

A query result

```sql
MY_DATABASE=# \copy (SELECT id, code FROM new_table) TO '/path/data.csv' CSV HEADER;
```

Or a table

```sql
MY_DATABASE=# \copy new_table TO '/path/data.csv' CSV HEADER;
```

Or with Tab delimiter

```sql
MY_DATABASE=# COPY new_table(id, code) FROM '/path/data.csv' DELIMITER E'\t' CSV HEADER;
```

### 5.5. List the available tables in the current `database`

Use the `\dt` command.

```bash
                              List of relations
 Schema |                        Name                        | Type  | Owner
--------+----------------------------------------------------+-------+-------
 public | account_account                                    | table | odoo
 public | account_account_account_tag                        | table | odoo
 public | account_account_financial_report                   | table | odoo
 public | account_account_financial_report_type              | table | odoo
 public | account_account_tag                                | table | odoo
 public | account_account_tag_account_tax_template_rel       | table | odoo
 public | account_account_tax_default_rel                    | table | odoo
 public | account_account_template                           | table | odoo
 public | account_account_template_account_tag               | table | odoo
 public | account_account_template_tax_rel                   | table | odoo
 ...
```

### 5.6. Describe a particular `table`

Use`\d table_name`

```bash
                                            Table "public.res_users"
      Column       |            Type             | Collation | Nullable |                Default
-------------------+-----------------------------+-----------+----------+---------------------------------------
 id                | integer                     |           | not null | nextval('res_users_id_seq'::regclass)
 active            | boolean                     |           |          | true
 login             | character varying           |           | not null |
 password          | character varying           |           |          |
 company_id        | integer                     |           | not null |
 partner_id        | integer                     |           | not null |
 create_date       | timestamp without time zone |           |          |
 share             | boolean                     |           |          |
 write_uid         | integer                     |           |          |
 create_uid        | integer                     |           |          |
 ...
```

## 6. Additional extra commands

### 6.1. Do you need to know all available commands?

Simple use: `\?`

```bash
General
  \copyright             show PostgreSQL usage and distribution terms
  \crosstabview [COLUMNS] execute query and display results in crosstab
  \errverbose            show most recent error message at maximum verbosity
  \g [FILE] or ;         execute query (and send results to file or |pipe)
  \gdesc                 describe result of query, without executing it
  \gexec                 execute query, then execute each value in its result
  \gset [PREFIX]         execute query and store results in psql variables
  \gx [FILE]             as \g, but forces expanded output mode
  \q                     quit psql
  \watch [SEC]           execute query every SEC seconds
  ...
```

### 6.2. You don't know the right syntax of PostgreSQL statements?

Lets know more about `DROP` statements. `\h DROP TABLE`

```bash
DOCKER_DB_SAMU=# \h DROP TABLE
Command:     DROP TABLE
Description: remove a table
Syntax:
DROP TABLE [ IF EXISTS ] name [, ...] [ CASCADE | RESTRICT ]

URL: https://www.postgresql.org/docs/12/sql-droptable.html
```

### 6.3. This is awesome, would you like to know the execution time of your queries?

Use `\timing` command. After that, execute a query and just watch!

```bash
DOCKER_DB_SAMU=# \timing
Timing is on.
DOCKER_DB_SAMU=# SELECT * FROM res_users;
Time: 187,940 ms
DOCKER_DB_SAMU=#
```

### 6.4. Did you know this one? `psql` + text editor

Use `\e` command to open the last executed command/query in a text editor.

```bash
DOCKER_DB_SAMU=# \e

Select an editor.  To change later, run 'select-editor'.
  1. /bin/nano        <---- easiest
  2. /usr/bin/nvim
  3. /usr/bin/vim.tiny
  4. /usr/bin/code
  5. /bin/ed

Choose 1-5 [1]: 2
```
