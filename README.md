# PostgreSQL

Most useful commands for PostgreSQL

### 1. Allow Remote Connections

Open `postgresql` configuration file located at:

```bash
# vim /etc/postgresql/12/main/postgresql.conf
```

And change:

```bash
listen_addresses = 'localhost'
```

To:

```bash
listen_addresses = '*'
```

Then edit the postgresql host based authentication (hba) file located at:

```bash
# vim /etc/postgresql/12/main/pg_hba.conf
```

And add the following lines in `# IPv4 local connections:` section.

```bash
host    all             all              0.0.0.0/0                       md5
host    all             all              ::/0                            md5
```

And finally, don't forget restart postgresql service:

```bash
# sudo systemctl restart postgresql
```

### 2. Create a `role` as `SUPERUSER`

First, change to `postgres` user after that enter to `psql` and execute de command

```bash
sudo su - postgres
postgres@DELL:~$
postgres@DELL:~$ psql
psql (12.4 (Ubuntu 12.4-0ubuntu0.20.04.1))
Type "help" for help.

postgres=#
postgres=# CREATE ROLE odoo WITH SUPERUSER CREATEDB CREATEROLE LOGIN ENCRYPTED PASSWORD 'odoo';
```

### 3. Drop `role`

```bash
# sudo su - postgres
# psql
postgres=# DROP ROLE odoo
```

#### Note:

ERROR: role "odoo" cannot be dropped because some objects depend on it
DETAIL: owner of database `MY_DATABASE`

So the reliable sequence of commands to drop a role is:

```bash
postgres=# REASSIGN OWNED BY odoo TO postgres; -- or some other trusted role
postgres=# DROP OWNED BY odoo; -- repeat both in ALL databases where the role owns anything or has any privileges!
postgres=# DROP USER odoo;
```

<!-- https://www.postgresql.org/docs/8.1/role-membership.html -->
<!-- https://www.postgresqltutorial.com/postgresql-alter-database/ -->

### 4. List existing roles

```bash
postgres=# SELECT rolname FROM pg_roles;
```

### 5. Create database

With previous created role `odoo`

```bash
$ sudo su - postgres
postgres@PC:~$ createdb MY_DATABASE --owner=odoo
```

Or whit in `psql` also with role `odoo`

```bash
postgresql=# CREATE DATABASE MU_DATABASE OWNER odoo;
```

Without role (by default owner is `postgres`)

```bash
$ sudo su - postgres
postgres@PC:~$ createdb MY_DATABASE
```

Or whit in `psql`, without role

```bash
postgresql=# CREATE DATABASE MY_DATABASE;
```

### 6. Change the owner of the database

```bash
postgres=# ALTER TABLE MY_DATABASE OWNER TO new_owner | current_user | session_user;
```

### 7. Drop database

```bash
$ sudo su - postgres
postgres@PC:~$ dropdb MY_DATABASE
```

### 8. Create table

```sql
postgres=# CREATE TABLE new_table (id varchar UNIQUE NOT NULL);
```

### 9. Alter table - ADD COLUMN

```sql
postgres=# ALTER TABLE new_table ADD COLUMN code varchar;
```

### 10. Import data in a given table

```sql
MY_DATABASE=# COPY new_table(id, code) FROM '/path/data.csv' DELIMITER ',' CSV HEADER;
```

### 11. Export data from a given table

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

### 12. Enlist the available databases

Use the `\l` command to get a list of all available databases.

```psql
                                        List of databases
         Name          |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
-----------------------+----------+----------+-------------+-------------+-----------------------
 DOCKER_DB_ODOOPROD    | odoo     | UTF8     | C           | en_US.UTF-8 |
 DOCKER_DB_SAMU        | odoo     | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 postgres              | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 template0             | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
                       |          |          |             |             | postgres=CTc/postgres
 template1             | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
                       |          |          |             |             | postgres=CTc/postgres
(5 rows)
```

### 13. Enlist the available tables in the current database

Use the `\dt` command.

```psql
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

### 14. Switch to another database

Use `\c DATABASE_NAME`

```psql
DOCKER_DB_SAMU=# \c DOCKER_DB_ODOOPROD
You are now connected to database "DOCKER_DB_ODOOPROD" as user "postgres".
DOCKER_DB_ODOOPROD=#
```

### 15. Describe a particular table

Use`\d table_name`

```psql
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
 action_id         | integer                     |           |          |
 write_date        | timestamp without time zone |           |          |
 signature         | text                        |           |          |
 password_crypt    | character varying           |           |          |
 alias_id          | integer                     |           |          |
 sale_team_id      | integer                     |           |          |
 is_guard_chief    | boolean                     |           |          |
 microred_id       | integer                     |           |          |
 department_id     | integer                     |           |          |
 cia_name          | character varying           |           |          |
 diresa_id         | integer                     |           |          |
 is_external_user  | boolean                     |           |          |
 red_id            | integer                     |           |          |
```

### 16. Which was the previous executed command?

`\g` it's your friend.

### 17. Do you need to know all available commands?

Simple use: `\?`

```psql
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

### 18. You don't know the right syntax of PostgreSQL statements?

Lets know more about `DROP` statements. `\h DROP TABLE`

```psql
DOCKER_DB_SAMU=# \h DROP TABLE
Command:     DROP TABLE
Description: remove a table
Syntax:
DROP TABLE [ IF EXISTS ] name [, ...] [ CASCADE | RESTRICT ]

URL: https://www.postgresql.org/docs/12/sql-droptable.html
```

### 19. This is awesome, would you like to know the execution time of your queries?

Use `\timing` command. After that, execute a query and just watch!

```psql
DOCKER_DB_SAMU=# \timing
Timing is on.
DOCKER_DB_SAMU=# SELECT * FROM res_users;
Time: 187,940 ms
DOCKER_DB_SAMU=#
```

### 20. Did you know this one? psql + text editor.

Use `\e` command to open the last executed command/query in a text editor.

```psql
DOCKER_DB_SAMU=# \e

Select an editor.  To change later, run 'select-editor'.
  1. /bin/nano        <---- easiest
  2. /usr/bin/nvim
  3. /usr/bin/vim.tiny
  4. /usr/bin/code
  5. /bin/ed

Choose 1-5 [1]: 2
```
