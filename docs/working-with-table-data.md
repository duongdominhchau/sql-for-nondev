# Working with table data

We learned how to create tables in the previous post. Now we will learn how to
put data into the table and read them out. First of all, let's create a table. I
will reuse the table in the previous post:

```sql
create table courses(
    name varchar,
    credits integer
);
```

There are 4 operations with the data, commonly abbreviated **CRUD** (**C**reate,
**R**ead, **U**pdate, **D**elete). We will learn them one by one.

## Reading from the table

We use `SELECT` to do this. Previously we learned how to use `SELECT` to
calculate expressions, now we are going to learn the `FROM` clause:

```sql
SELECT <column-or-expression-1>, <column-or-expression-2>, ...
FROM <table>;
```

You can use `*` as a shorthand for selecting all columns.

Example:

```sql
select credits, name from courses;
select name, credits from courses;
select * from courses;
```

To restrict the result to only some rows, add a `WHERE` clause with the filter
condition. Rows matching the conditions will be returned.

```sql
-- Find all courses with 2 or more credits
select *
from courses
where credits > 1;
```

!!! note

    For other types, the comparisons are trivial. However, for boolean type, you
    need to use `IS` instead of `=` (`value IS TRUE` instead of `value = TRUE`). The
    subtle difference will be explained in a future post.

When we only want to get unique rows, we can add `DISTINCT` after `SELECT`. If
we have `phone_numbers` table like this:

| name | phone_number |
| ---- | ------------ |
| A    | 0123456789   |
| A    | 0222344545   |
| B    | 092145536    |

then

```sql
SELECT name
FROM phone_numbers;
```

produces

| name |
| ---- |
| A    |
| A    |
| B    |

while

```sql
SELECT *
FROM phone_numbers;
```

produces

| name |
| ---- |
| A    |
| B    |

## Inserting new rows

`INSERT INTO` creates new rows in the table.

```sql
INSERT INTO <table>
VALUES(value_for_column_1, value_for_column_2, ...);
```

Example:

```sql
insert into courses
values('SQL 101', 2);
```

We can also explicitly list the columns to insert, this allow us to ignore the
column order and list the values in the order we want:

```sql
insert into courses(credit, name)
values(2, 'SQL 101');
```

You can see that I flipped the values, but it still work fine, because the
columns name are also listed in reverse order after the table name.

## Updating rows

```sql
UPDATE <table>
SET
    <column1> = <new-value1>,
    <column2> = <new-value2>,
    ...
WHERE <conditions>;
```

This finds all rows matching the conditions and change the values of the columns
listed.

Example:

```sql
update courses
set credits = 1
where name = 'SQL 101';
```

## Deleting rows

```sql
DELETE FROM <table>
WHERE <conditions>;
```

This will find rows matching the conditions and delete them.

Example:

```sql
-- Delete all courses with 2 credits
delete from courses
where credits = 2;
```

!!! danger

    Without a condition, it will **delete all rows**.

## Exercises

Given this table:

```sql
create table users(
    full_name varchar,
    username varchar,
    email varchar,
    active boolean
);
```

Write SQL queries to:

- Find the emails of inactive accounts

??? Answer

    ```sql
    select email from users where active is false;
    ```

- Check if user `admin` exists in the table. Hint: find the corresponding row
    and return anything. If no row returned, the user does not exist.

??? answer

    ```sql
    select 1 -- or `select 2`, `select *`, `select username`, anything works
    from users
    where username = 'admin';
    ```

- Add a new **active** user with `username`, `full_name` and `email` of your
    choice.

??? answer

    ```sql
    insert into users(username, full_name, email, active)
    values ('sql101', 'SQL 101', 'sql101@example.com', true)
    ```

- Change the `full_name` of the account added above to `ABC`

??? answer

    We need to find the row to update.

    - The best choice is `username`, because it is designed to uniquely identify a
        user and is quite stable
    - The next option is `email`, it is also unique in this case, but usually we
        will need to expand the design to allow multiple addresses, so it is good
        but not as good as `username`.
    - Matching by current name also works if you are lucky, but it is unreliable
        (different people with the same name is common) and should be avoided.
    - `active` is something you should never use to uniquely identify a row. With 3
        users, you are guaranteed to find two rows with the same `active` value.

    Therefore, this is the best answer:

    ```sql
    update users
    set full_name = 'ABC'
    where username = 'sql101';
    ```

    And this is another acceptable one:

    ```sql
    update users
    set full_name = 'ABC'
    where email = 'sql101@example.com';
    ```

- Delete the user you added above

??? answer

    ```sql
    delete from users
    where username = 'sql101'
    ```
