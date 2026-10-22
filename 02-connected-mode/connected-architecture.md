# Connected Architecture

## Definition

In connected mode, your app keeps the connection to the database **open** while it reads or changes data.

## Why do we use it?

- Changes reach the database immediately.
- It is the best choice for DML operations (INSERT, UPDATE, DELETE).
- It is the fastest way to read data once, from start to end.

## How does it work?

```
1. Define the connection
2. Define the command
3. Open the connection
4. Execute the command
5. Use the results (for example, show them in a control)
6. Close the connection
```

Main classes:

| Class | Job |
|---|---|
| `SqlConnection` | Connects to the database |
| `SqlCommand` | Holds the SQL query and runs it |
| `SqlDataReader` | Reads the rows returned by a SELECT |

## Important notes

- One connection can run more than one command, but **one after another**.
- You cannot open a second `SqlDataReader` on the same connection while the first one is still open (unless MARS is enabled).
- Always close the connection. The easiest way is a `using` block. See [Connection](connection-and-connection-string.md).

## Next

[Connection and Connection String](connection-and-connection-string.md)