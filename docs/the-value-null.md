# The value NULL

`NULL` is a special value, it marks the absent of value. If you have a score of
`0`, it means you already have a score, and the score is `0`. However, if you
have a score of NULL, it means you have no score yet. When NULL is involved, it
changes how things work.

## NULL in expression

When `NULL` is used where a condition is expected, it will be treated similar to
`false`. For example:

```sql
select * from users where null;
```

Because `null` is falsey, the condition is always considered false and no row is
included in the result.

In expressions, `NULL` causes the whole expression evaluated to `NULL`:

```sql
select 1 + NULL * 3 / 4 != 2; -- Result: NULL
```

This is what happened to the first query:

- `1 + NULL` evaluates to `NULL`, so we have `NULL * 3 / 4 != 2`
- `NULL * 3` evaluates to `NULL` too, now we have `NULL / 4 != 2`
- `NULL / 4` evaluates to `NULL`, the expression becomes `NULL != 2`
- `NULL != 2` evaluates to `NULL`

This also happens with string concatenation:

```sql
select 'a' || null; -- Result: NULL
```

And even list inclusion check can be affected:

```
select 1 IN (1, 2, 3, NULL); -- Result: true
select 1 NOT IN (1, 2, 3, NULL); -- Result: false, all are good until now, but...

select 4 IN (1, 2, 3, NULL); -- Result: NULL
select 4 NOT IN (1, 2, 3, NULL); -- Result: NULL
select NULL IN (1, 2, 3, NULL); -- Result: NULL
select NULL NOT IN (1, 2, 3, NULL); -- Result: NULL
```

A single `NULL` can ruin the whole expression, this is why we should avoid
`NULL` whenever possible.

## The `IS` operator

Unlike `=`, `IS` can correctly perform the test when `NULL` is involved. If the
value can be `NULL`, we must use this to handle it properly

```sql
select null is null; -- result: 1 row containing true because `null is null` is evaluated to `true`
select null = null; -- result: no row because null = null is evaluated to `null`
```

It also works correctly when we check for true/false:

```sql
select null is true; -- Result: false
select null is false; -- Result: false
```

## `NOT NULL` column

By default, the columns are nullable. When you use `INSERT INTO` without
specifying a value for the column, it will receive `NULL` as value.

```sql
create table nullable_column_example(
    a varchar,
    b varchar
);

insert into nullable_column_example(a) values('1');
-- The line above is the same as 
-- insert into nullable_column_example(a, b) values('1', null);

-- Query it out and see for yourself
select * from nullable_column_example;
```

To prevent `NULL`, we can mark a column as `NOT NULL`, the database will prevent
inserting rows with `NULL` value for that column.

!!! tip

    Because `NOT NULL` column is more restricted than nullable column, when we
    create a new column, we should always try to make it non-nullable. Moving from
    non-null to null is easy, the other direction is much harder because we have to
    deal with existing `NULL` values first before we can mark the column as
    `NOT NULL`.

To mark a column as `NOT NULL`, add `NOT NULL` after the column type when we
create the table:

```sql
create table example(required_column varchar not null, optional_column varchar);
```

Now when we insert NULL into the table, we get an error. Try it yourself:

```sql
insert into example(required_column) values(null);
```

You should see the error:

```plain
Failed to run sql query: ERROR:  23502: null value in column "required_column" of relation "example" violates not-null constraint

DETAIL:  Failing row contains (null).
```

That's the database helping us prevent bad data when we tell them what we
expect.

When you want to change an existing column to `NOT NULL`, use `ALTER TABLE`:

```sql
alter table example
alter column optional_column set not null;
```

To make it nullable, we use `DROP` instead of `SET`

```sql
alter table example
alter column optional_column drop not null;
```

Now try inserting a `NULL` value then change the column to `NOT NULL` and see
what happens:

```sql
insert into example(required_column, optional_column)
values('a', null);

alter table example
alter column optional_column set not null;
```

You should see the error

```plain
Error: Failed to run sql query: ERROR: 23502: column "optional_column" of relation "example" contains null values
```

This is why we should start with `NOT NULL` if possible.

## Null-related functions

- `COALESCE(value1, value2, ...)`: returns the first non-null value

    ```sql
    select COALESCE(null, null, 1); -- Result: 1
    ```

- `NULLIF(value, null_marker)` returns `NULL` if `value` equals `null_marker`,
    otherwise it returns the value. This is useful when we have special marker
    for `NULL` like `'(none)'` or `''` (empty string).

    ```sql
    select NULLIF('a', ''), NULLIF('(none)', '(none)'); -- Result: 'a', NULL
    ```

## Exercises

1. Write a query to create a new table named `products` with a non-null column
    `name` and 2 optional (nullable) columns `email`, `phone`.

    ??? Answer

        ```sql
        create table customers(
            name varchar not null,
            email varchar,
            phone varchar
        );
        ```

2. Insert some data into the table above using this script:

    ```sql
    insert into customers(name, email, phone)
    values
        ('Klein', 'klein.moretti@lotm.com', null),
        ('John', null, '0123456789'),
        ('Merlin', null, null);
    ```

    Now, find the contact of each customer. If `phone` is available, use it,
    otherwise use `email`, and if there is none, return `'N/A'`

    ??? Answer

        ```sql
        select name, coalesce(phone, email, 'N/A') contact
        from customers;
        ```

3. Reimplement `NULLIF` using `CASE`. Hint: Remember that using `=` with `NULL`
    results in `NULL`.

    ??? Answer

        We need to consider these cases:

        | `a`    | `b`    | `NULLIF(a, b)` |
        | ------ | ------ | -------------- |
        | `NULL` | `NULL` | `NULL`         |
        | `NULL` | `1`    | `NULL`         |
        | `1`    | `NULL` | `1`            |
        | `1`    | `1`    | `NULL`         |
        | `1`    | `2`    | `1`            |

        We can see that `NULL` makes the calculation more complex, so we should
        prioritize handling it first. This is one possible solution:

        ```sql
        NULLIF(a, b)
        ```

        is equivalent to

        ```sql
        CASE
            WHEN a IS NULL THEN NULL
            WHEN b IS NULL THEN a -- This is only considered when a is not null
            WHEN a = b THEN NULL
            ELSE a
        END
        ```
