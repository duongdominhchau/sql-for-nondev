# Creating tables

## `CREATE TABLE` query syntax

To create a new table, we use the `CREATE TABLE` query. This is the minimal form
of `CREATE TABLE`:

```sql
CREATE TABLE <table_name>(
    column1 <column1-type>
);
```

## Data types

Every column must have a data type to describe what data it is going to hold.
For now, let's stick with the basic ones:

- `varchar`: arbitrary text. Values of this type need to be wrapped inside
    single quote. Example: `'this is a string'`

- `integer`: whole number, like `-999`, `0`, `123`

- `numeric(precision, scale)`: exact real number (e.g: `0.123`) with predefined
    precision and scale. Take up more space and is slower to process, but is
    more reliable. `precision` is the number of digits allowed, `scale` is the
    number of digits allowed after the decimal point. Example:

    ```plain
    vvvv v precision of 5
    1234.5
         ^ scale of 1
    ```

- `float`: inexact real number, the precision is dynamic depending on current
    value, faster to process and take up less space. **DO NOT USE THIS
    CORRECTNESS IS IMPORTANT**

- `boolean`: hold the value`true` or `false`. This is usually stored as number
    (`0` as `false`, `1` as `true`)

- `timestamptz` (short for `timestamp with timezone`): datetime stored in UTC.
    When read out, it is converted back to local timezone.

!!! info

    There are other types with the same characteristics as the types above, but with
    different size. For example:

    | Type       | Size    | Minimum value              | Maximum value             |
    | ---------- | ------- | -------------------------- | ------------------------- |
    | `smallint` | 2 bytes | -32,768                    | 32,767                    |
    | `integer`  | 4 bytes | -2,147,483,648             | 2,147,483,647             |
    | `bigint`   | 8 bytes | -9,223,372,036,854,775,808 | 9,223,372,036,854,775,807 |

    They are for optimizing storage, so let the engineers remember them, we don't
    need to care.

## Designing tables

You need to determine what to store and find the appropriate name for them, then
determine the best type for each column. Choosing correct types makes working
with the value easier.

!!! info

    An example of what happens when we choose wrong type: Everything can be stored
    as `varchar`, but if we store numbers as `varchar`, we can't multiply two
    numbers together, because the varchar type also include values like 'abc' which
    cannot be multiplied. Besides, `'1.0'` is considered different from `'1'`.

    This is the common theme in software engineering: the more restrictions imposed,
    the more helpful the computer. Each restriction provides information to the
    computer and help it make better decisions.

    Try it yourself: Look at this and guess what is the value of the empty cell?

    ![](./sudoku-near-complete.png)

    Did you come up with `7`? Now think about how you arrived at that number. Why
    can't it be `2`, or `X`, or `@` but `7`? You must have realized that it is the
    only missing number from 1 to 9. The rule of sudoku (any row, column, and
    bordered 3x3 square must contain numbers from 1 to 9) mandates it to be `7` and
    cannot be anything else. That rule is the restriction, and with the restriction
    it allows us to infer new information. Type system works the same way.

Let's say I want to track the number of credits for each course, I can create a
table like this:

```sql
create table courses(
    name varchar,
    credits integer
);
```

By convention, we use plural form for table name because a table will contain
many records. So, if you want a database table to store points of interest, you
would name it `points_of_interest`.

## Destroying existing tables

To delete an existing table, use

```sql
DROP TABLE <table_name>;
```

That's it!

## Exercises

!!! question

    What is the smallest `numeric` type I can use to store 999999.12345?

??? Answers

    There are 5 digits after the decimal points, so the scale is 5. There are 6
    digits before, so the precision is 6 + 5 = 11. We need at least `numeric(11, 5)`
    to store this value

!!! question

    Write an SQL query to create a table to store reviews. We need to store author,
    rating score (5-star rating, can select half star) and the comment content. Also
    track when the review is made.

??? answer

    ```sql
    create table reviews(
        author varchar,
        rating numeric(2,1),
        content varchar,
        created_at timestamptz
    );
    ```

    Your column names may be different, that's fine. Something like this is also
    acceptable:

    ```sql
    create table reviews(
        author_name varchar,
        rating_score numeric,
        comment varchar,
        created_at timestamptz
    );
    ```

    The important part is everyone agree on the same word usage.

!!! question

    Delete the tables created in the exercise above

??? Answer

    ```sql
    drop table reviews;
    ```
