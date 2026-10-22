# Connected vs Disconnected

Both models work with the same database. The difference is **when** the connection is open and **where** your changes live.

## Comparison

| | Connected | Disconnected |
|---|---|---|
| Connection | Open while you work | Open only to load and to save |
| Main classes | `SqlConnection`, `SqlCommand`, `SqlDataReader` | `SqlDataAdapter`, `DataTable`, `DataSet` |
| Changes | Go to the database immediately | Stay in memory until `adapter.Update()` |
| Reading data | Forward-only, read-only | Can move in any direction and edit |
| Memory use | Low (one row at a time) | Higher (the data is stored in memory) |
| Data freshness | Always current | A copy, may become old |

## When to use each one

**Use connected mode when:**

- You run DML commands (INSERT, UPDATE, DELETE) and the change must be saved now. Example: a bank transfer.
- You read a lot of data once, from start to end, and want it fast.
- You need one value, like `COUNT(*)`.

**Use disconnected mode when:**

- The user edits many rows and saves them together.
- You need to use the same data more than once without querying again.
- You work with several related tables in a `DataSet`.

## About "which one is better for SELECT?"

For a simple read, `SqlDataReader` (connected) is the **fastest** option, because it does not store all the rows. Disconnected mode is better for SELECT when you need to **keep** the data and work on it, not when you only need to read it once.

## Hybrid

> **TODO:** Add a short example of using both models in one app. Waiting for the original meaning of "hybrid" from the course notes.

## Next

Back to the [README](README.md)