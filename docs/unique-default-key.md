# Unique, default, and key

## Unique column

In previous posts, we have the users table like this:

```sql
create table users(
    username varchar not null,
    full_name varchar,
    created_at timestamptz
);
```

We assume `username` is unique, but there is nothing in the schema to tell the
database to enforce this, so:

```sql
insert into users(username)
values ('a'), ('a');
```

will happily add two rows with the same username into the database. To enforce
the uniqueness, we need to add a `UNIQUE` constraint for that column. All
constraints (like `NOT NULL`) are written after the type. The constraints order
are not important so `NOT NULL UNIQUE` and `UNIQUE NOT NULL` is the same.

Adding the constraint to the `CREATE TABLE` query, we have:

```sql
create table users(
    username varchar not null unique,
    full_name varchar,
    created_at timestamptz
);
```

Now try inserting duplicated values again:

```sql
insert into users(username)
values ('a'), ('a');
```

You should now see the error

```plain
Failed to run sql query: ERROR:  23505: duplicate key value violates unique constraint "users_username_key"

DETAIL:  Key (username)=(a) already exists.
```

## Default value

In the `users` table above, we always want `created_at` to be current time. With
the current schema, we need to explicitly pass `NOW()` to `INSERT INTO`. To make
this easier, SQL allows us to specify column default value using
`DEFAULT <value>`.

Because we are going to set a default value for it, and a `NULL` value for
`created_at` is not useful, we should also add `NOT NULL`.

```sql
create table users(
    username varchar not null unique,
    full_name varchar,
    created_at timestamptz not null default now()
);
```

Now, when we skip the `created_at` column when `INSERT`, it will receive the
value of `NOW()`:

```sql
insert into users(username)
values ('a');

select * from users;
```

## Primary key

When you update or delete a row, you need to write a condition that matches
exactly that row and nothing more. The columns used to uniquely identify a row
is called a **key**.

You can have many keys, but can choose only one to use with SQL, that key will
be called **primary key**. To define a single-column primary key, add
`PRIMARY KEY` constraint to that column. **Primary key = unique + not null**, so
you don't need to repeat the other two.

In our `users` table, `username` is used for uniquely identifying rows, and it
is not null, so it is a good candidate to use as primary key.

```sql
create table users(
    username varchar primary key,
    full_name varchar,
    created_at timestamptz not null default now()
);
```

You can also declare the primary key separately after columns:

```sql
create table users(
    username varchar,
    full_name varchar,
    created_at timestamptz not null default now(),
    primary key(username)
);
```

This is one line longer, but can work with multi-column primary key.

!!! note

    Every table must have a primary key. In previous posts I skipped that to keep
    the content focused. From now on you need to always choose a primary key for
    your new tables.

## Foreign key

Primary key is the identity of a row. Now that every table has a primary key, we
can refer to a row on another table.This forms a **relationship** and the column
used to reference another row is called **foreign key**. To tell the database
about this relationship, we add `REFERENCES table(column)` constraint to the
foreign key column.

!!! note

    We can have as many foreign keys as needed. Besides, a column can serve as
    primary key of this table while also reference column on another table (i.e:
    foreign key).

Let's imagine that we want to store payment methods added to a user account, we
can write it like this:

```sql
create table payment_methods(
    username varchar not null references users(username),
    method varchar not null
);
```

With this constraint, when we insert payment method for nonexistent user, it
will error:

```sql
insert into payment_methods(username, method)
values ('nonexistent', 'PayPal');
```

```plain
Failed to run sql query: ERROR:  23503: insert or update on table "payment_methods" violates foreign key constraint "payment_methods_username_fkey"

DETAIL:  Key (username)=(nonexistent) is not present in table "users".
```

## Synthetic key

`username` is a good primary key because it is not null and unique, this is
called **natural key** because the key contains data but also happens to satisfy
the primary key condition. However, `username` can change, and changing key
means changing foreign keys too. In some cases, it's hard to find a column like
`username` to use as primary key, so people come up with **synthetic key**.

A synthetic key is a column not containing any real data. We fill it with
anything we want as long as they are unique. The most common way to fill it is
with auto-increment sequence. Another strategy uses random value like UUID v4.

In the `users` table above, we should add a synthetic key and use it as primary
key.

```sql
create table users(
    id integer generated always as identity primary key,
    username varchar not null unique,
    full_name varchar,
    created_at timestamptz not null default now()
);
```

The `GENERATED ALWAYS AS IDENTITY` part means the database will automatically
choose a value for that column, we can't provide value for it. If you want the
database to choose a value but still allow us to override it, use
`GENERATED BY DEFAULT AS IDENTITY` instead.

Now try it:

```sql
insert into users(username)
values ('user_a'), ('user_b');

select * from users;
```

You should see the `id` receive an auto-increment value without us doing
anything.

## Exercises

Design tables for storing courses information and student enrolments. Here's the
columns, determine what constraints to add:

```sql
create table courses(
    id integer,
    name varchar,
    credits integer,
    created_at timestamptz
);

create table enrolments(
    id integer,
    course_id integer,
    student_name varchar
);
```

??? Answer

    ```sql
    create table courses(
        id integer primary key generated always as identity,
        name varchar not null,
        credits integer not null,
        created_at timestamptz not null default now()
    );

    create table enrolments(
        -- Intentionally flipped order to show that
        -- constraints order does not matter
        id integer generated always as identity primary key,
        course_id integer not null references courses(id),
        student_name varchar not null
    );
    ```
