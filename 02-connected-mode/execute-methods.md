# Execute Methods

> **Scope:** .NET Framework ADO.NET. Differences in modern .NET are marked.
> **Sources:** [ExecuteReader](https://learn.microsoft.com/en-us/dotnet/api/system.data.sqlclient.sqlcommand.executereader), [ExecuteScalar](https://learn.microsoft.com/en-us/dotnet/api/system.data.sqlclient.sqlcommand.executescalar), [ExecuteNonQuery](https://learn.microsoft.com/en-us/dotnet/api/system.data.sqlclient.sqlcommand.executenonquery), [ExecuteXmlReader](https://learn.microsoft.com/en-us/dotnet/api/system.data.sqlclient.sqlcommand.executexmlreader)

---

## Definition

`SqlCommand` only **describes** a command. An **Execute method** sends it to SQL Server. Choose the method by what you expect **back**.

| Method | Returns | Use it for |
|---|---|---|
| `ExecuteReader()` | `SqlDataReader` | SELECT with many rows |
| `ExecuteScalar()` | `object` (one value) | SELECT with one value, like `COUNT(*)` |
| `ExecuteNonQuery()` | `int` (rows affected) | INSERT, UPDATE, DELETE, and DDL (CREATE TABLE...) |
| `ExecuteXmlReader()` | `XmlReader` | `SELECT ... FOR XML` queries (`SqlCommand` only) |

All of them need an **open connection**.

### Which one do I use?

```
Need rows to read?          -> ExecuteReader
Need one single value?      -> ExecuteScalar
Change data / no result?    -> ExecuteNonQuery
Need the result as XML?     -> ExecuteXmlReader
```

---

## 1. ExecuteReader

Returns a `SqlDataReader` to read rows forward-only. Full details in `sqldatareader.md`.

```csharp
using (var con = new SqlConnection(cs))
using (var cmd = new SqlCommand("SELECT St_Id, St_Fname FROM Student", con))
{
    con.Open();
    using (SqlDataReader reader = cmd.ExecuteReader())
    {
        while (reader.Read())
            Console.WriteLine($"{reader.GetInt32(0)} - {reader.GetString(1)}");
    }
}
```

### `CommandBehavior` overload

`ExecuteReader(CommandBehavior)` changes how the reader behaves. Two to know now:

| Value | Meaning |
|---|---|
| `CloseConnection` | The connection closes when the reader is closed |
| `SequentialAccess` | For large values and binary data. Without it, an `OutOfMemoryException` can occur and the connection is closed |

```csharp
using (SqlDataReader reader = cmd.ExecuteReader(CommandBehavior.CloseConnection))
{
    // read...
}   // closing the reader also closes the connection
```

---

## 2. ExecuteScalar

Returns the **first column of the first row**. Extra columns and rows are **ignored**.

```csharp
using (var con = new SqlConnection(cs))
using (var cmd = new SqlCommand("SELECT COUNT(*) FROM Student", con))
{
    con.Open();
    int count = (int)cmd.ExecuteScalar();
}
```

### Check for null

| Situation | `ExecuteScalar` returns |
|---|---|
| Query returns no rows | `null` |
| First cell is NULL in the database | `DBNull.Value` |

```csharp
object result = cmd.ExecuteScalar();

if (result == null || result == DBNull.Value)
    Console.WriteLine("No value");
else
    Console.WriteLine(result);
```

### Get the new identity after an INSERT

```csharp
string sql = "INSERT INTO Student (St_Fname) VALUES (@name); " +
             "SELECT CAST(scope_identity() AS int)";

using (var con = new SqlConnection(cs))
using (var cmd = new SqlCommand(sql, con))
{
    cmd.Parameters.AddWithValue("@name", "Aya");   // parameters: next file
    con.Open();
    int newId = (int)cmd.ExecuteScalar();
}
```

This pattern comes from the Microsoft example for `ExecuteScalar`.

---

## 3. ExecuteNonQuery

Runs a command that does not return rows. Returns the **number of rows affected**.

```csharp
using (var con = new SqlConnection(cs))
using (var cmd = new SqlCommand(
    "UPDATE Student SET St_Fname = @name WHERE St_Id = @id", con))
{
    cmd.Parameters.AddWithValue("@name", "Ahmed");
    cmd.Parameters.AddWithValue("@id", 5);

    con.Open();
    int rows = cmd.ExecuteNonQuery();   // how many rows changed
}
```

### When does it return `-1`?

| Case | Return value |
|---|---|
| UPDATE / INSERT / DELETE | Rows affected |
| Any other statement (SELECT, CREATE TABLE...) | `-1` |
| Rollback happened | `-1` |
| `SET NOCOUNT ON` is active | The statement's count is not included, so it can return `-1` |
| Table has a **trigger** | The count **includes** rows affected by the trigger |

Also: output parameters and return values mapped to parameters are **still filled**, even though no rows are returned.

---

## 4. ExecuteXmlReader

Reads `FOR XML` results as an `XmlReader` (`using System.Xml;`).

```csharp
using (var con = new SqlConnection(cs))
using (var cmd = new SqlCommand("SELECT St_Id, St_Fname FROM Student FOR XML AUTO", con))
{
    con.Open();
    using (XmlReader xr = cmd.ExecuteXmlReader())
    {
        xr.Read();
        while (xr.ReadState != ReadState.EndOfFile)
            Console.WriteLine(xr.ReadOuterXml());
    }
}
```

Why not `ExecuteReader`? With `ExecuteReader`, SQL Server returns XML longer than 2,033 characters split across **multiple rows** of 2,033 characters. Microsoft says to use `ExecuteXmlReader` for `FOR XML` queries.

---

## Async versions

Each method has an async version (for example, `ExecuteReaderAsync`). See `05-advanced-topics/asynchronous-ado-net.md`.

---

## Modern .NET differences

- Same four methods and same return values.
- For new SQL Server apps, use `Microsoft.Data.SqlClient`.

---

## Common mistakes

1. Using `ExecuteNonQuery` for a SELECT. It returns `-1`, not data.
2. Casting `ExecuteScalar()` without checking `null` / `DBNull.Value`.
3. A wrong cast: the cast must match the SQL type. `COUNT(*)` returns `int`, as in the Microsoft example.
4. Using `ExecuteReader` when only one value is needed.
5. Forgetting `con.Open()`.
6. Trusting `ExecuteNonQuery`'s return value when `SET NOCOUNT ON` is used.
7. Concatenating user input into SQL (use parameters).

---

## Summary

- `ExecuteReader` = rows. `ExecuteScalar` = one value. `ExecuteNonQuery` = rows affected. `ExecuteXmlReader` = XML.
- `ExecuteScalar` returns `null` for no rows and `DBNull.Value` for a NULL cell.
- `ExecuteNonQuery` returns `-1` for statements that are not INSERT/UPDATE/DELETE, for rollbacks, and when `NOCOUNT` hides the count.

---

## Next

[Parameters and SQL Injection](parameters-and-sql-injection.md)