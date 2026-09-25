# D-1: Importing CSV Data

Continuing from setting up the database, let's look at importing data. Before importing data, we need a table to store it. While we should normally account for data types, for this first attempt, we'll keep it simple and use `text` for everything.

### 1. Create the table

We'll define the table based on the CSV headers.

```sql
create schema if not exists import;
drop table if exists import.master_plan;

create table import.master_plan(
    start_time_utc text,
    duration text,
    date text,
    team text,
    spass_type text,
    target text,
    request_name text,
    library_definition text,
    title text,
    description text
);
```

### 2. Prepare the COPY command

Now, define the SQL query to perform the actual data import:

```sql
COPY import.master_plan
FROM '/absolute/path/to/master_plan.csv'
WITH DELIMITER ',' HEADER CSV;
```

### 3. Execution

Wrap these commands into a single file, e.g., `build.sql`, and execute it from your shell:

```bash
psql -d <your_db_name> -f build.sql
```

### 4. Troubleshooting Permissions

If that doesn't work (as it didn't for me), you might run into PostgreSQL and Linux permission issues.

*   **PostgreSQL permissions:** You may need to grant the PostgreSQL user permissions to read client files.
*   **Linux permissions:** PostgreSQL runs as its own user (`postgres`). Because it is considered an 'other' user, it cannot traverse your home directory by default. To allow the PostgreSQL application to read files within your home directory, edit the service configuration:

```bash
sudo systemctl edit postgresql
```

Then, set `ProtectHome=read-only` in the override configuration.
