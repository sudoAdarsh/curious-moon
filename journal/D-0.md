# Curious Moon

Hello this is journal for myself and to any reader who reads this, i will try to jolt down points i find intresting and usefull cuz i have knack for forgetting stuff :)

## D-Day

Let's begin with installation, i am on arch os so will/can only tell aout this one but core philospohy is same everyhwere ig

### 1. Install PostgreSQL

```bash
sudo pacman -S postgresql
```

### 2. Initialize the PostgreSQL database cluster

```bash
sudo -iu postgres initdb --locale=C.UTF-8 --encoding=UTF8 -D /var/lib/postgres/data
```

### 3. Start PostgreSQL

```bash
sudo systemctl enable --now postgresql
```

### 4. Create a PostgreSQL user and local database

Create a PostgreSQL user with the same name as your Linux user, then create a database owned by that user:

```bash
sudo -u postgres createuser "$USER"
sudo -u postgres createdb "$USER"
```

Connect to PostgreSQL:

```bash
psql
```

### 5. Verify

Inside `psql`:

```sql
\conninfo
```

You should be connected to a database named after your Linux username.
