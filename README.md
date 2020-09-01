# PostgreSQL
Very usefull commands for PostgreSQL

## PostgreSQL: Allow Remote Connections

```bash
# vim /etc/postgresql/12/main/postgresql.conf

listen_addresses = '*'

# sudo systemctl restart postgresql
```

```bash
# vim /etc/postgresql/12/main/pg_hba.conf

host    all             all              0.0.0.0/0                       md5
host    all             all              ::/0                            md5
```

## Create odoo role as SUPERUSER in PostgreSQL

```bash
# sudo su - postgres
postgres=# CREATE ROLE odoo WITH SUPERUSER CREATEDB CREATEROLE LOGIN ENCRYPTED PASSWORD 'odoo';
```

## Create database

```bash
sudo su - postgres
createdb DATABASE
psql
```

Or with owner

```bash
createdb DATABASE --owner=odoo
```

## Create table

```sql
postgres=# CREATE TABLE new_table (id varchar UNIQUE NOT NULL);
```

## Alter table - ADD COLUMN

```sql
postgres=# ALTER TABLE new_table ADD COLUMN code varchar;
```

## Import data in a given table

```sql
DATABASE=# COPY new_table(id, code) FROM '/path/data.csv' DELIMITER ',' CSV HEADER;
```

## Export data from a given table

A query result

```sql
DATABASE=# \copy (SELECT id, code FROM new_table) TO '/path/data.csv' CSV HEADER;
```

Or a table

```sql
DATABASE=# \copy new_table TO '/path/data.csv' CSV HEADER;
```

Or with Tab delimiter

```sql
DATABASE=# COPY new_table(id, code) FROM '/path/data.csv' DELIMITER E'\t' CSV HEADER;
```

## Enlist the available databases

Use the `\l` command to get a list of all available databases.

## Enlist the available tables in the current database

Use the `\dt` command.

## Switch to another database

Use `\c DATABASE_NAME`

## Describe a particular table

Use`\d table_name`

## Dam, which was the previous executed command?

`\g` it's your friend.

## Do you need to know all available commands?

Simple use: `\?`

## You don't know the right syntax of PostgreSQL statements?

Lets know more about `DROP` statements. `\h DROP TABLE`

## This is awesome, would you like to know the execution time of your queries?

Use `\timing` command. After that, execute a query and just watch!

## Did you know this one? psql + text editor.

Use `\e` command to open the last executed command/query in a text editor.

