# Relationships and table joining

We learned about foreign key in the previous post, I mentioned that it is used
for setting up **relationships**. In this post, we will learn about the kinds of
relationships and how to make use of them.

## Relationships

When a table refers to or being referred by another table, we say they have a
relationship. Relationships are everywhere. When we buy something, we become the
buyer of that thing, that's a relationship. When we enroll into a course, we are
establishing a relationship with the course.

In SQL, relationships are represented using foreign keys. This ensures you don't
establish relationship between an existing record and a nonexistent one.

```sql
create customers(
    id integer generated always as identity primary key,
    name varchar not null
);

create table purchases(
    id integer generated always as identity primary key,

    -- This establish the relationship
    customer_id integer not null references customers(id)
);
```

In the database world, we care primarily about the cardinality of relationship,
in simpler words: how many records are involved. When a student enrolls into a
course:

- A student can enroll into many courses
- A course can have many students

This is called many-to-many relationship.
