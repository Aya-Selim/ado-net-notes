# SqlCommand

> **Scope:** .NET Framework ADO.NET. Differences in modern .NET are marked.
> **Sources:** [SqlCommand](https://learn.microsoft.com/en-us/dotnet/api/system.data.sqlclient.sqlcommand), [CommandType enum](https://learn.microsoft.com/dotnet/api/system.data.commandtype), [CommandTimeout](https://learn.microsoft.com/dotnet/api/system.data.sqlclient.sqlcommand.commandtimeout)

---

## Definition

`SqlCommand` holds the SQL statement (or stored procedure name) you want to run, and the connection to run it on.

It only **describes** the command. To run it, you call an **Execute method** (see `execute-methods.md`).

---

## Main properties

| Property | Meaning | Default |
|---|---|---|
| `CommandText` | The SQL text or the stored procedure name | empty string |
| `CommandType` | How `CommandText` is interpreted: `Text` or `StoredProcedure` | `Text` |
| `Connection` | The `SqlConnection` to use | `null` (unless set in the constructor) |
| `Parameters` | Values sent to the query. See `parameters-and-sql-injection.md` | empty |
| `CommandTimeout` | Seconds to wait for the command to run | `30` |
| `Transaction` | The `SqlTransaction` the command runs in. See `sql-transactions.md` | none |

### `CommandType` values

| Value | Meaning | SqlClient |
|---|---|---|
| `Text` | A SQL statement (default) | Supported |
| `StoredProcedure` | The name of a stored procedure | Supported |
| `TableDirect` | A table name | **Not for SqlClient.** Microsoft says it is only supported by the OLE DB provider |

`CommandType` is in the `System.Data` namespace, so add `using System.Data;`.

---

## Ways to create a command

```csharp
// 1. Text + connection (most common)
var cmd = new SqlCommand("SELECT St_Id, St_Fname FROM Student", con);

// 2. From the connection
SqlCommand cmd2 = con.CreateCommand();
cmd2.CommandText = "SELECT St_Id, St_Fname FROM Student";

// 3. Property by property (long way)
var cmd3 = new SqlCommand();
cmd3.CommandText = "SELECT St_Id, St_Fname FROM Student";
cmd3.CommandType = CommandType.Text;   // default
cmd3.Connection = con;
```

Constructors available: `SqlCommand()`, `SqlCommand(text)`, `SqlCommand(text, connection)`, `SqlCommand(text, connection, transaction)`.

---

## Full example

```csharp
using System.Data;
using Microsoft.Data.SqlClient;

using (var con = new SqlConnection(cs))
using (var cmd = new SqlCommand("SELECT St_Id, St_Fname FROM Student", con))
{
    con.Open();   // the connection must be open before executing

    using (SqlDataReader reader = cmd.ExecuteReader())
    {
        while (reader.Read())
            Console.WriteLine($"{reader.GetInt32(0)} - {reader.GetString(1)}");
    }
}
```

`SqlCommand` is disposable too. Put it in a `using` block.

---

## Stored procedure

```csharp
using (var cmd = new SqlCommand("GetAllStudents", con))
{
    cmd.CommandType = CommandType.StoredProcedure;   // CommandText = procedure name only
    // add parameters here if the procedure needs them
}
```

With `CommandType.StoredProcedure`, put only the **procedure name** in `CommandText`. Details: `stored-procedures.md`.

---

## CommandTimeout

```csharp
cmd.CommandTimeout = 60;   // seconds
```

- Default is **30 seconds**. If the command takes longer, it fails with an error.
- `0` means **no limit**: it waits forever. Avoid it.
- A negative value is not allowed.
- This is **not** the same as the connection timeout (time to open a connection). See `timeouts.md`.

---

## Do not build SQL from user input

```csharp
// BAD: SQL injection risk
var bad = new SqlCommand("SELECT * FROM Student WHERE St_Fname = '" + name + "'", con);
```

Use parameters instead (see `parameters-and-sql-injection.md`).

---

## Modern .NET differences

- Same properties and same behavior.
- For new SQL Server apps, use `Microsoft.Data.SqlClient` (`using Microsoft.Data.SqlClient;`) instead of `System.Data.SqlClient`.

---

## Common mistakes

1. Forgetting to set the `Connection` property.
2. Forgetting `con.Open()` before executing.
3. Forgetting `CommandType.StoredProcedure` when `CommandText` is a procedure name.
4. Using `CommandType.TableDirect` with `SqlCommand`.
5. Setting `CommandTimeout = 0` and letting a bad query hang forever.
6. Concatenating user input into `CommandText`.
7. Not disposing the command.

---

## Summary

- `SqlCommand` = SQL text (or procedure name) + connection (+ parameters, timeout, transaction).
- `CommandType`: `Text` (default) or `StoredProcedure` for SqlClient.
- `CommandTimeout` default is 30 seconds. Use parameters, not string concatenation.

---

## Next

[Execute Methods](execute-methods.md)