# D-1: Defining Schema and Table Structure

Today I learned how to create a more robust database structure. I moved away from just dumping everything as text and started defining proper data types for columns.

I also learned about the importance of schemas in PostgreSQL. They act as namespaces, which helps avoid name collisions and organizes tables by their purpose (like keeping import tables separate).

I finally figured out how to use the `SERIAL` type combined with `PRIMARY KEY`. This creates an auto-incrementing integer key automatically. It's a handy shortcut that PostgreSQL provides, managing sequences and constraints under the hood.

Here is the refined SQL for creating the table:

```sql
create schema if not exists import;
drop table if exists import.master_plan;

create table import.master_plan(
  id serial primary key,
  the_date date,
  title varchar(100),
  description text
);
```

To execute this script, I can use the `-f` flag with `psql`:

```bash
psql enceladus -f build.sql
```
