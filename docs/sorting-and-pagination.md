# Sorting and pagination

This post uses this sample table:

```sql
create table users(
    id integer primary key generated always as identity,
    name varchar not null,
    behavior_score integer,
    active boolean not null default true,
    country varchar
);
insert into users(name, active, behavior_score, country)
values
    ('Liam Johnson', true, 100, 'United States'),
    ('Emma Tremblay', true, 50, 'Canada'),
    ('Noah Müller', false, 100, 'Germany'),
    ('Yuto Sato', true, 75, 'Japan'),
    ('Olivia Smith', true, 10, 'United States'),
    ('Lucas Roy', false, 1, 'Canada'),
    ('Mia Schmidt', true, 100, 'Germany'),
    ('Mei Tanaka', true, 52, 'Japan'),
    ('Ethan Davis', false, 97, 'United States'),
    ('Sophia Wagner', true, 66, 'Germany');
```

## Sorting with `ORDER BY`

To sort the result, we can list the columns used for sorting after `ORDER BY`:

```sql
select *
from users
order by name;
```

Add `DESC` after the column name to sort that column descending, the default is
ascending sort.

```sql
select *
from users
order by name desc;
```

!!! note

    `ORDER BY` cannot appear before `WHERE`

We can list multiple columns after `ORDER BY`, the first column will be used for
sorting, then when two rows having the same value, it considers the second
column and similarly for the remaining columns. Example:

```sql
select *
from users
order by country, name;
```

!!! warning

    If multiple rows have the same value for all columns listed in ORDER BY, their
    order are undefined and can vary between runs. In the first run it may be sorted
    before, but in the next run it may be sorted after.

## Pagination

We can use `LIMIT` to restrict the number of records to return and `OFFSET` to
specify where to start reading. Offset is the distance between a row and the
first row, so offset `0` reads from the beginning while offset `3` skips 3 first
rows before reading.

!!! note

    The keyword `LIMIT` is non-standard, but it is too common, so I have to show you
    this instead of the longer form defined by SQL standard.

```sql
select *
from users
order by country, name
limit 5 offset 0;
```

Try changing that offset from 0 to 1 and see what happens.

??? note "How does it work?"

    First it sorts the table due to the presence of `ORDER BY`:

    | Read cursor | id  | name          | behavior_score | active | country       |
    | ----------- | --- | ------------- | -------------- | ------ | ------------- |
    | --->        | 2   | Emma Tremblay | 50             | true   | Canada        |
    |             | 6   | Lucas Roy     | 1              | false  | Canada        |
    |             | 7   | Mia Schmidt   | 100            | true   | Germany       |
    |             | 3   | Noah Müller   | 100            | false  | Germany       |
    |             | 10  | Sophia Wagner | 66             | true   | Germany       |
    |             | 8   | Mei Tanaka    | 52             | true   | Japan         |
    |             | 4   | Yuto Sato     | 75             | true   | Japan         |
    |             | 9   | Ethan Davis   | 97             | false  | United States |
    |             | 1   | Liam Johnson  | 100            | true   | United States |
    |             | 5   | Olivia Smith  | 10             | true   | United States |

    The read cursor is pointing to the first row, now it advances by the `OFFSET`
    number we specified. Let's say we use `OFFSET 1`, it will move the read cursor
    to the next row:

    | Read cursor | id  | name          | behavior_score | active | country       |
    | ----------- | --- | ------------- | -------------- | ------ | ------------- |
    |             | 2   | Emma Tremblay | 50             | true   | Canada        |
    | --->        | 6   | Lucas Roy     | 1              | false  | Canada        |
    |             | 7   | Mia Schmidt   | 100            | true   | Germany       |
    |             | 3   | Noah Müller   | 100            | false  | Germany       |
    |             | 10  | Sophia Wagner | 66             | true   | Germany       |
    |             | 8   | Mei Tanaka    | 52             | true   | Japan         |
    |             | 4   | Yuto Sato     | 75             | true   | Japan         |
    |             | 9   | Ethan Davis   | 97             | false  | United States |
    |             | 1   | Liam Johnson  | 100            | true   | United States |
    |             | 5   | Olivia Smith  | 10             | true   | United States |

    Now it starts reading from the row pointed by the read cursor. The number of
    rows to read is specified by the `LIMIT` clause: 5 rows. We get:

    | Read cursor | id  | name          | behavior_score | active | country |
    | ----------- | --- | ------------- | -------------- | ------ | ------- |
    |             | 6   | Lucas Roy     | 1              | false  | Canada  |
    |             | 7   | Mia Schmidt   | 100            | true   | Germany |
    |             | 3   | Noah Müller   | 100            | false  | Germany |
    |             | 10  | Sophia Wagner | 66             | true   | Germany |
    |             | 8   | Mei Tanaka    | 52             | true   | Japan   |

    Then it discards the rest because it reached the limit we set.

We can read page by page using `LIMIT` and `OFFSET`. Let's say we want to read
all data but with 20 rows each time, we can:

- Read the first 20 rows with `LIMIT 20 OFFSET 0`
- Read the next 20 rows with `LIMIT 20 OFFSET 20`
- Read the next 20 rows with `LIMIT 20 OFFSET 40`
- ... and repeat that until the number of rows returned is fewer than 20. That's
    the sign we reached the last page and there is no more record to read.

!!! danger

    Although not syntactically required, pagination always need to have sorting
    applied first. When the order is undefined, you can't know what will be read out
    when `LIMIT` is applied.

## Exercises

- List all users, sort by behavior_score descending and name ascending when the
    score is equal.

??? Answer

    ```sql
    select *
    from users
    order by behavior_score desc, name;
    ```

- Find the top 5 users with highest `behavior_score`, use the same sorting as
    the previous exercise

??? Answer

    ```sql
    select *
    from users
    order by behavior_score desc, name
    limit 5;
    ```

- Find the user with third-highest behavior_score from Germany

??? Answer

    ```sql
    select *
    from users
    where country = 'Germany'
    order by behavior_score desc, name
    limit 1 offset 2;
    ```
