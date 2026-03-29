# Functions and operators

In this post we look at more advanced processing of values via functions and
operators.

`+`, `-`, `*`, `/` on integers are called operators. Operators are what we can
do with the value of a specific type. Functions are technically the same as
operators, but they are named using words instead of symbols, for example:
`sin`, `cos`, `abs`, etc.

## Logical operators

These operators work with boolean values and produce another boolean.

`AND`: `false` if at least one side is `false`, or `true` if all sides are
`true`. Choose whatever sounds more natural to you, they are the same.

| `a`   | `b`   | `a AND b` |
| ----- | ----- | --------- |
| false | false | false     |
| false | true  | false     |
| true  | false | false     |
| true  | true  | true      |

`OR`: `true`if at least one side is`true`

| `a`   | `b`   | `a OR b` |
| ----- | ----- | -------- |
| false | false | false    |
| false | true  | true     |
| true  | false | false    |
| true  | true  | true     |

`NOT` flips the value:

| `a`   | `NOT a` |
| ----- | ------- |
| false | true    |
| true  | false   |

## Comparison

For comparison, we have `>` (greater), `>=` (greater or equal), `<` (less than),
`<=` (less than or equal), `=` (equal), `!=` (or `<>`, both are not equal, but
`!=` is more common in programming languages). The result is a boolean.

| `a` | `b` | `a < b` | `a <= b` | `a > b` | `a >= b` | `a == b` | `a != b` |
| --- | --- | ------- | -------- | ------- | -------- | -------- | -------- |
| 1   | 2   | true    | true     | false   | false    | false    | true     |
| 1   | 1   | false   | true     | false   | true     | true     | false    |

We also have some English comparisons:

- `x BETWEEN a AND b`: same as `a <= x AND x <= b` but sounds more like natural
    language.
- `x NOT BETWEEN a AND b`: same as `NOT (a <= x AND x <= b)`

## Arithmetic

`+` (add), `-` (subtract), `*` (multiply), `/` (divide), `%` (remainder), `^`
exponent. Example:

```sql
select 2 + 3 add, 2 - 3 sub, 2 * 3 mul, 2 / 4 div, 2 % 4 mod, 2 ^ 4 exp;
```

Result:

| add | sub | mul | div | mod | exp |
| --- | --- | --- | --- | --- | --- |
| 5   | -1  | 6   | 0   | 2   | 16  |

Some functions are also commonly used and worth mentioning here.

- `sqrt`: Square root

- `abs`: Value without negative sign

    ```sql
    select abs(-1) a, abs(0) b, abs(1) c;
    ```

    | a   | b   | c   |
    | --- | --- | --- |
    | 1   | 0   | 1   |

- `ceil`: **Round up** to the nearest integer not smaller than current value

    ```sql
    select ceil(1) a, ceil(-1.5) b, ceil(1.5) c;
    ```

    | a   | b   | c   |
    | --- | --- | --- |
    | 1   | -1  | 2   |

- `floor`: **Round down** to the nearest integer not greater than current value

    ```sql
    select floor(1) a, floor(-1.5) b, floor(1.5) c;
    ```

    | a   | b   | c   |
    | --- | --- | --- |
    | 1   | -2  | 1   |

- `round`: Round to nearest integer (can be up or down depending on the value)

    ```sql
    select round(1) a, round(-1.5) b, round(1.5) c;
    ```

    | a   | b   | c   |
    | --- | --- | --- |
    | 1   | -2  | 2   |

    When using with real numbers, you can also specify how many digits after the
    decimal point to keep:

    ```sql
    -- Keep only 2 digits after the decimal point
    select round(1.2345, 2);
    ```

- `trunc`: Discard the fractional part (the digits after decimal point).

    ```sql
    select trunc(1) a, trunc(-1.5) b, trunc(1.5) c;
    ```

    | a   | b   | c   |
    | --- | --- | --- |
    | 1   | -1  | 1   |

See full list at <https://www.postgresql.org/docs/current/functions-math.html>

## String functions

For `varchar` type, we don't have `+`, `-`, `*`, `/` or `AND`, `OR`, `NOT`, but
we still have an operator: `||`. This is called concatenation and is used to
join two strings into a single string. Example:

```sql
select 'a' || 'b' as result;
```

Result:

| result |
| ------ |
| ab     |

Besides, we also have lots of functions to work with strings:

- `length(s)`: return the number of characters in the string `s`

    ```sql
    select length('A B  C '); -- Note the trailing whitespace
    -- Result: 7   ^^^^^^^
    ```

- `lower(s)`: convert the string `s` to lowercase

    ```sql
    select lower('AbC');
    -- Result: 'abc'
    ```

- `upper(s)`: convert the string `s` to lowercase

    ```sql
    select lower('AbC');
    -- Result: 'ABC'
    ```

- `concat_ws(separator, s1, s2, ...)`: concatenate the strings s1, s2, ...
    together and separate the using the `separator`.

    ```sql
    select concat_ws('and', 'cat', 'dog');
    -- Result: 'catanddog'
    select concat_ws(' and ', 'cat', 'dog');
    -- Result: 'cat and dog'
    ```

See full list at <https://www.postgresql.org/docs/current/functions-string.html>

## Pattern matching

We also have `LIKE` for pattern matching. `s LIKE pattern` returns true if the
string `s` matches the provided `pattern`. `LIKE` is more powerful than `=`
because it has 2 characters with special meaning: `_` means match exactly 1
character and `%` means match 0 or more characters. Example:

```sql
select 'abc' LIKE '_';
-- Result: false. Because 'abc' are 3 characters, while the '_' pattern means
-- match a string containing a single character
```

```sql
select 'abc' LIKE 'a_c';
-- Result: true. `a` matches `a`, `b` matches `_`, `c` matches `c`
```

```sql
select 'abc' LIKE '%';
-- Result: true. % matches 0 or more, so when standing alone, it can match any string
```

`%` is more useful. We can use it to check if a string starts or ends with
something.

- `value LIKE '%s'` checks if `value` ends with `'s'`

```sql
select 'abc' LIKE '%c';
-- Result: true, `ab` matches `%`, `c` matches `c`
```

- `value LIKE 's%'` checks if `value` starts with `'s'`

```sql
select 'abc' LIKE '%c';
-- Result: true, `ab` matches `%`, `c` matches `c`
```

- `value LIKE '%s%'` checks if `value` contains `'s'`

```sql
select 'abc' LIKE '%c%';
-- Result: true, `ab` matches `%`, `c` matches `c`, `%` matches the empty string
```

We also have an even more powerful feature for text matching: regular
expression, but that's complex, let's skip it for now.

## Datetime

Just remember that we can use `NOW()` or `CURRENT_TIMESTAMP()` (they are the
same) to get current datetime. The rest can be looked up when we need. See
<https://www.postgresql.org/docs/current/functions-datetime.html>

## List inclusion

To test if a value match any of a list, we can use `IN` operator:

```sql
select 1 in (1, 2, 3); -- Result: true
select 4 in (1, 2, 3); -- Result: false
select 'a' in ('a', 'b', 'c'); -- Result: true
select 'A' in ('a', 'b', 'c'); -- Result: false
```

`NOT IN` can be used for the opposite test.

## Conditional expression

```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ...
    ELSE result_n
END;
```

The `ELSE` clause is optional, and you can have as many `WHEN` clauses as you
want. It will match from top to bottom and stop at the first matched condition.
This is like nested `IF` in Excel but in a more readable form.

Example:

```sql
select
    case
        when 1 > 1 then '1 > 1'
        when 1 < 2 then '1 < 2'
        else 'don''t know'  -- Double ' to represent a literal single quote inside a string
    end;
```

## Exercises

Given this table:

```sql
create table users(
    first_name varchar,
    last_name varchar,
    username varchar,
    created_at timestamptz
);

insert into users(first_name, last_name, username)
values
    ('Fors', 'Wall', 'fors.wall'),
    ('Leonard', 'Mitchell', 'thestar');
```

Write SQL queries to

- Add a new user and set `created_at` to current time.

??? Answer

    ```sql
    insert into users(first_name, last_name, username, created_at)
    values('Audrey', 'Hall', 'audrey.hall', NOW());
    ```

- Query full name (first name followed by last name) of all users.

??? Answer

    ```sql
    select concat_ws(' ', first_name, last_name) full_name
    from users;
    ```

- Query users with a dot in their username

??? Answer

    ```sql
    select *
    from users
    where username LIKE '%.%';
    ```

- List all username with an extra column named `username_has_dot`. This column
    contains `'Y'` if the username column has any dot character (`.`) inside and
    `'N'` otherwise.

??? Answer

    ```sql
    select
        username,
        case
            when username LIKE '%.%' then 'Y'
            else 'N'
        end
    from users;
    ```
