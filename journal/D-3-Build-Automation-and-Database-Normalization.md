# D-3: Build Automation with Make and Database Normalization

### 1. Automating with Make

`Make` is a pre-historic build tool, and its essence is simple: turning one thing into another.

In a `Makefile`, we define **targets**, which represent the processes or files we want to build. The actual build happens through a **recipe**, which is basically a shell command. If target `X` needs to be built before target `Y`, we specify `X` as a **prerequisite** for target `Y`.

Our `Makefile` needs to achieve two main targets:
1. Create the `import` schema, load the `master_plan` table, and create the `events` table.
2. Normalize and import the raw data into the `events` table.

### 2. Normalization

Normalization is a concept used to reduce data repetition and save disk space by removing ambiguity and creating predictable tables. It improves both storage efficiency and compute speed—for instance, comparing integers is much faster than comparing strings.

Our raw `master_plan` table has quite a lot of repeated text information. To make it more efficient, let's create a central table called `events`:

```sql
create table events(
    id serial primary key,
    time_stamp timestamptz not null,
    title varchar(500),
    description text,
    event_type_id int,
    spass_type_id int,
    target_id int,
    team_id int,
    request_id int
);
```

### 3. Importing Data into the `events` Table (Data Cleaning Lesson)

Before importing, it's always a smart move to inspect the data and verify which columns and data types we can actually trust. For example, `master_plan` contains a date of `29-02-2014`—which is clearly a leap-year error on JPL's system side (2014 wasn't a leap year!).

To avoid dealing with broken date strings, we can rely on the `start_time_utc` column and cast it directly to `timestamptz`:

```sql
insert into events(
    time_stamp,
    title,
    description
)
select
    import.master_plan.start_time_utc::timestamptz at time zone 'UTC',
    import.master_plan.title,
    import.master_plan.description
from import.master_plan;
```

### 4. Extracting Lookup Tables

Next, we need to extract the unique values for each categorical column into its own lookup table. We can do this using the `DISTINCT` keyword in a `SELECT` query and pipe the result directly into a new table using `INTO`—no need to run `CREATE TABLE` beforehand:

```sql
select distinct(team)
as description
into teams
from import.master_plan;
```

Now our `teams` table holds all the unique teams from `master_plan`. Let's add a primary key so we can relate these lookup IDs back to the `events` table:

```sql
alter table teams
add id serial primary key;
```

We repeat this exact pattern for the remaining lookup tables (`requests`, `targets`, `spass_types`, and `event_types`).

### 5. Completing the `events` Table with Foreign Keys & Joins

Now that we have separate lookup tables, we can enforce referential integrity using **foreign keys** (`REFERENCES`). Let's update our `CREATE TABLE` statement for `events`:

```sql
drop table if exists events;
create table events(
    id serial primary key,
    time_stamp timestamptz not null,
    title varchar(500),
    description text,
    event_type_id int references event_types(id),
    target_id int references targets(id),
    team_id int references teams(id),
    request_id int references requests(id),
    spass_type_id int references spass_types(id)
);
```

And likewise, update our `INSERT` statement to pull the foreign key IDs by joining against our new lookup tables:

```sql
insert into events(
    time_stamp,
    title,
    description,
    event_type_id,
    target_id,
    team_id,
    request_id,
    spass_type_id
)
select 
    import.master_plan.start_time_utc::timestamp,
    import.master_plan.title,
    import.master_plan.description,
    event_types.id as event_type_id,
    targets.id as target_id,
    teams.id as team_id,
    requests.id as request_id,
    spass_types.id as spass_type_id
from import.master_plan
left join event_types
    on event_types.description = import.master_plan.library_definition
left join targets
    on targets.description = import.master_plan.target
left join teams
    on teams.description = import.master_plan.team
left join requests
    on requests.description = import.master_plan.request_name
left join spass_types
    on spass_types.description = import.master_plan.spass_type;
```

With this, we've abstracted three separate time/date columns into a single `timestamptz` column and converted repeated string-based entries into compact integer foreign keys!

### 6. Quick Note to Self: Types of SQL Joins

While putting this together, I learned the four basic types of joins (using `LEFT JOIN` above so we never accidentally drop an event if one of its lookup fields is `NULL`):

1. **Inner Join (`INNER JOIN`):** Only returns rows where matching data is present in *both* tables.
2. **Left Join (`LEFT JOIN`):** Keeps all rows from the left table and pads unmatched columns from the right table with `NULL`.
3. **Right Join (`RIGHT JOIN`):** Same as a left join, just mirrored (keeps all rows from the right table, pads the left with `NULL`).
4. **Full Outer Join (`FULL OUTER JOIN`):** Keeps all rows from *both* tables, padding with `NULL` on whichever side is missing a match.
