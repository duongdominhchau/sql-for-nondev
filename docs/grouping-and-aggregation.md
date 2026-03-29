# Grouping and aggregation

Up until now, we only use `SELECT` with per-row operations. However, there are
many useful operations that work on group of rows. We will learn about them in
this post.

First, we need to prepare our table for demonstration:

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

## Grouping

When we use `SELECT`, we can specify column(s) used for grouping using
`GROUP BY`, rows with the same values for these columns will be put into the
same group.

Unlike our previous queries, query with `GROUP BY` does not return rows, it
returns groups, but groups can't be the result of a query, so we need a way to
convert the groups into a rows. If we just select everything like this it won't
work:

```sql
SELECT *
FROM users
GROUP BY country;
```

This is because there can be two rows with the same country. A group can only be
converted to a row, so if there are two different rows with the same country,
which one will be returned? Both `Emma Tremblay` and `Lucas Roy` are from
`Canada`, there is no reason why we return one instead of the other. This is
unclear to the database, so it rejects our query. If we change the query to

```sql
SELECT country
FROM users
GROUP BY country;
```

You won't see the error. This is because although there are multiple rows, they
all have the same `country`, so there is no different.

!!! tip

    Anything listed in `GROUP BY` can be `SELECT`ed directly.

    You can also notice that the output is not duplicated, so when using `SELECT a`
    together with `GROUP BY a` it behaves effectively like `SELECT DISTINCT`.

So, what do we do about the non-grouped columns? That's where we need
aggregation.

## Aggregation

Aggregation is a function that operates on the whole group and reduce all these
values into a single one. For example: `SUM` takes in a list of numbers, then
return a single value calculated by adding all the input numbers. That's an
aggregation.

Common aggregation functions are:

- `MAX`: return the largest value
- `MIN`: return the smallest value
- `SUM`: add all input together
- `AVG`: `SUM` divides number of inputs
- `COUNT`: count number of input rows.

When there can be different values for a column in a group, we cannot select
them directly, but we can select the aggregation of that column:

```sql
-- Count number of users from each country
select country, count(*)
from users
group by country;
```

We are not limited to a single aggregation:

```sql
-- Find lowest and highest behavior_score per country
select country, max(behavior_score), min(behavior_score)
from users
group by country;
```

!!! note

    If no `GROUP BY` is used, the whole table is considered one group.

`COUNT` can filter before counting using `FILTER` clause like this:

```sql
select count(*) filter(where country = 'Canada')
from users;
```

This will loop through the whole table and filter for rows with
`country = 'Canada'`, then count the number of rows found.

## Group filtering with `HAVING`

When we query for rows, we can filter using WHERE, but with `GROUP BY`, we need
another keyword: `HAVING`. Where is used for row-wise comparison, while `HAVING`
filters the whole group so it takes in aggregation comparison. Example:

```sql
-- find country with more than 2 users
select country, count(*)
from users
group by country
having count(*) > 2; 
```

## Window function

!!! warning

    This is advanced topic, feel free to skip.

`WHERE` for row-level filtering, `HAVING` for group-level filter, what if we
need to filter at both levels? Or we want to filter at group level but also want
to take out the value of every row in the group? Enter **window function**.
Window function allows us to perform aggregation calculation as it walks through
each row. This opens up a lot of possibilities.

With `PARTITION BY`, it will perform group calculation and put the result into
each row, so we have both the aggregated value and the original row:

```sql
select
    country,
    name,
    behavior_score,
    sum(behavior_score) over (partition by country) accumulated_score
from users;
```

Without `PARTITION BY`, it will treat all rows to be in the same group and
aggregates over them.

We can also specify the order within the group with `ORDER BY`. When `ORDER BY`
is used, the aggregation is calculated differently:

```sql
select
    country,
    name,
    behavior_score,
    sum(behavior_score) over (partition by country order by name desc) accumulated_score
from users;
```

If you look careful at the output, there's some interesting behavior here:

| country       | name          | behavior_score | accumulated_score |
| ------------- | ------------- | -------------- | ----------------- |
| Canada        | Lucas Roy     | 1              | 1                 |
| Canada        | Emma Tremblay | 50             | 51                |
| Germany       | Sophia Wagner | 66             | 66                |
| Germany       | Noah Müller   | 100            | 166               |
| Germany       | Mia Schmidt   | 100            | 266               |
| Japan         | Yuto Sato     | 75             | 75                |
| Japan         | Mei Tanaka    | 52             | 127               |
| United States | Olivia Smith  | 10             | 10                |
| United States | Liam Johnson  | 100            | 110               |
| United States | Ethan Davis   | 97             | 207               |

The `accumulated_score` is not the same for users from the same country! It only
`SUM()` the values of rows before or at current row in the group.

??? info "How does it work?"

    First we have the whole table

    | id  | name          | behavior_score | active | country       |
    | --- | ------------- | -------------- | ------ | ------------- |
    | 1   | Liam Johnson  | 100            | true   | United States |
    | 2   | Emma Tremblay | 50             | true   | Canada        |
    | 3   | Noah Müller   | 100            | false  | Germany       |
    | 4   | Yuto Sato     | 75             | true   | Japan         |
    | 5   | Olivia Smith  | 10             | true   | United States |
    | 6   | Lucas Roy     | 1              | false  | Canada        |
    | 7   | Mia Schmidt   | 100            | true   | Germany       |
    | 8   | Mei Tanaka    | 52             | true   | Japan         |
    | 9   | Ethan Davis   | 97             | false  | United States |
    | 10  | Sophia Wagner | 66             | true   | Germany       |

    Then the `PARTITION BY` divides this into groups:

    | id  | name         | behavior_score | active | country       |
    | --- | ------------ | -------------- | ------ | ------------- |
    | 1   | Liam Johnson | 100            | true   | United States |
    | 5   | Olivia Smith | 10             | true   | United States |
    | 9   | Ethan Davis  | 97             | false  | United States |

    | id  | name          | behavior_score | active | country |
    | --- | ------------- | -------------- | ------ | ------- |
    | 2   | Emma Tremblay | 50             | true   | Canada  |
    | 6   | Lucas Roy     | 1              | false  | Canada  |

    | id  | name          | behavior_score | active | country |
    | --- | ------------- | -------------- | ------ | ------- |
    | 3   | Noah Müller   | 100            | false  | Germany |
    | 7   | Mia Schmidt   | 100            | true   | Germany |
    | 10  | Sophia Wagner | 66             | true   | Germany |

    | id  | name       | behavior_score | active | country |
    | --- | ---------- | -------------- | ------ | ------- |
    | 4   | Yuto Sato  | 75             | true   | Japan   |
    | 8   | Mei Tanaka | 52             | true   | Japan   |

    Then each group is sorted by the `ORDER BY`:

    | id  | name         | behavior_score | active | country       |
    | --- | ------------ | -------------- | ------ | ------------- |
    | 5   | Olivia Smith | 10             | true   | United States |
    | 1   | Liam Johnson | 100            | true   | United States |
    | 9   | Ethan Davis  | 97             | false  | United States |

    | id  | name          | behavior_score | active | country |
    | --- | ------------- | -------------- | ------ | ------- |
    | 6   | Lucas Roy     | 1              | false  | Canada  |
    | 2   | Emma Tremblay | 50             | true   | Canada  |

    | id  | name          | behavior_score | active | country |
    | --- | ------------- | -------------- | ------ | ------- |
    | 10  | Sophia Wagner | 66             | true   | Germany |
    | 3   | Noah Müller   | 100            | false  | Germany |
    | 7   | Mia Schmidt   | 100            | true   | Germany |

    | id  | name       | behavior_score | active | country |
    | --- | ---------- | -------------- | ------ | ------- |
    | 4   | Yuto Sato  | 75             | true   | Japan   |
    | 8   | Mei Tanaka | 52             | true   | Japan   |

    Then it runs the aggregation for each group from top to bottom and add the
    result to the row. I'll skip irrelevant columns to keep this focused

    United States:

    | Current row | name         | behavior_score | accumulated_score |
    | ----------- | ------------ | -------------- | ----------------- |
    | --->        | Olivia Smith | 10             | 10                |
    |             | Liam Johnson | 100            |                   |
    |             | Ethan Davis  | 97             |                   |

    | Current row | name         | behavior_score | accumulated_score |
    | ----------- | ------------ | -------------- | ----------------- |
    |             | Olivia Smith | 10             | 10                |
    | --->        | Liam Johnson | 100            | 110               |
    |             | Ethan Davis  | 97             |                   |

    | Current row | name         | behavior_score | accumulated_score |
    | ----------- | ------------ | -------------- | ----------------- |
    |             | Olivia Smith | 10             | 10                |
    |             | Liam Johnson | 100            | 110               |
    | --->        | Ethan Davis  | 97             | 207               |

    The same for the remaining 3 groups. Finally, `SELECT` takes the values out and
    return to us.

We can also get the current row within the group using `row_number()`.

```sql
select
    country,
    name,
    row_number() over (partition by country order by name asc) row
from users
```

When combined this with other advanced features like subqueries/common table
expression, a lot of new possibilities open. We can calculate running total
using this, we can pick out top-K from each groups, we can count how many
records are below or above average.

These features are advanced features, so don't worry if you don't understand
them right now.

The downside is window function is quite complex to understand, and its support
is not good either.

## Exercises

- Count number of inactive users and number of active users. Need to show both
    `id` and `name`

??? Answer

    ```sql
    select
        count(*) filter (where active is false) num_inactive,
        count(*) filter (where active is true) num_active
    from users;
    ```

- Calculate the inactive percentage (inactive users / total users * 100) per
    country.

    Because `COUNT` returns an integer, you will need to convert it to numeric
    first, otherwise the database will perform integer division and truncate the
    fractional part. Use `CAST(COUNT(...) AS numeric)` instead of just
    `COUNT(...)`. Also remember to round to 2 digits after the decimal point.

??? Answer

    ```sql
    select
        country,
        round(
            cast(count(*) filter (where active is false) as numeric) / count(*) * 100,
            2
        ) inactive_percentage
    from users
    group by country;
    ```
