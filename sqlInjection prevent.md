Why $1 and $2?

This:

VALUES ($1, $2)

with:

[name, city]

is a parameterized query.

It is safer than directly putting user input into SQL:

User Input
    ↓
Parameterized Query
    ↓
PostgreSQL

It helps prevent SQL injection.
