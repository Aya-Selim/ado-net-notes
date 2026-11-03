# Connected Architecture

> **Scope:** .NET Framework ADO.NET.
> **Sources:** [ADO.NET Architecture](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/ado-net-architecture), [Retrieving and Modifying Data in ADO.NET](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/retrieving-and-modifying-data)

---

## Definition

In connected mode, your app keeps the connection to the database **open** while it reads or changes data.

## Why use it?

- Commands run **directly** against the database.
- Good for **direct commands**: INSERT, UPDATE, DELETE, stored procedures.
- `SqlDataReader` is the **fastest** way to read data once, forward-only and read-only. It uses less memory than a `DataSet`.

## Main classes

| Class | Job |
|---|---|
| `SqlConnection` | Connects to the database |
| `SqlCommand` | Holds the SQL (or stored procedure name) and runs it |
| `SqlDataReader` | Reads the rows returned by a SELECT |

## How it works

```
1. Create the connection
2. Create the command
3. Open the connection   (as late as possible)
4. Execute the command
5. Use the results
6. Close the connection  (as early as possible, use `using`)
```

## Code examples

### Read data

```csharp
using (var connection = new SqlConnection(connectionString))
using (var command = new SqlCommand("SELECT Id, Name FROM Students", connection))
{
    connection.Open();

    using (SqlDataReader reader = command.ExecuteReader())
    {
        while (reader.Read())
            Console.WriteLine($"{reader.GetInt32(0)} - {reader.GetString(1)}");
    }
}   // connection is closed here
```

### Change data

```csharp
using (var connection = new SqlConnection(connectionString))
using (var command = new SqlCommand(
    "UPDATE Students SET IsActive = 1 WHERE Id = 1", connection))
{
    connection.Open();
    int rowsAffected = command.ExecuteNonQuery();
}
```

> Hard-coded SQL is only for learning. Real code must use parameters (see `parameters-and-sql-injection.md`).

## Important notes

- One connection can run many commands, but **one after another**.
- You cannot open a second `SqlDataReader` on the same connection while the first is still open, unless **MARS** is enabled (see `05-advanced-topics/mars.md`).
- While a reader is open, the connection is busy. Keep the work inside the loop **short**.
- Always close the connection. The easiest way is a `using` block (see `resource-management-and-using.md`).

## Modern .NET

Same concept and same flow. For new SQL Server apps use `Microsoft.Data.SqlClient` instead of `System.Data.SqlClient`.

## Common mistakes

1. Forgetting `connection.Open()`.
2. Not closing or disposing the connection, command, or reader.
3. Opening a second reader on the same connection (without MARS).
4. Doing slow work inside the `while (reader.Read())` loop, which keeps the connection busy.

## Summary

- Connected = connection stays open while you work.
- Flow: connection -> command -> open -> execute -> use results -> close.
- Best for fast reads and direct commands. Disconnected mode is better for caching and editing data.

---

## Next

[Connection and Connection String](connection-and-connection-string.md)