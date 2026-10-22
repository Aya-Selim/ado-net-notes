# Disconnected Architecture

## Definition

In disconnected mode, you load data from the database into memory, close the connection, and work on the in-memory copy. Later, you send the changes back to the database.

## Why do we use it?

- You do not keep a connection open while the user works. This saves database resources.
- You can change many rows in memory, then save them all together.
- You can keep data in memory and use it more than once without querying again.

## How does it work?

**Load:**

```
1. Define the connection
2. Define the command (SELECT)
3. Create a SqlDataAdapter
4. Fill a DataTable or DataSet
```

**Change:** insert, update, or delete rows in the DataTable. Nothing changes in the database yet.

**Save:**

```
1. Define the Insert, Update, and Delete commands
2. Give them to the adapter
3. Call adapter.Update()
```

Main classes:

| Class | Job |
|---|---|
| `DataTable` | One table in memory (rows and columns) |
| `DataSet` | A group of DataTables, with the relations between them |
| `SqlDataAdapter` | The bridge between the database and the DataTable or DataSet |

## Important notes

- Changes in memory are **not** in the database until you call `adapter.Update()`.
- Because the data is a copy, it can become old. Another user may change the same rows in the database while you are working. This is called a **concurrency** problem.

## Next

[DataTable and DataSet](datatable-and-dataset.md)