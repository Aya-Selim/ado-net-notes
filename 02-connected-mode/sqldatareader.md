# SqlDataReader

> **Scope:** .NET Framework ADO.NET. Differences in modern .NET are marked.
> **Sources:** [Retrieving Data Using a DataReader (.NET Framework)](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/retrieving-data-using-a-datareader), [Retrieve data by a DataReader (SqlClient)](https://learn.microsoft.com/en-us/sql/connect/ado-net/retrieve-data-by-datareader), [SqlDataReader class](https://learn.microsoft.com/en-us/dotnet/api/system.data.sqlclient.sqldatareader)

---

## Definition

`SqlDataReader` reads the rows returned by a query, **one row at a time**, as an **unbuffered stream** of data.

## Why use it?

- Very **fast** and uses **little memory**. Rows are not cached in memory.
- A good choice for large amounts of data.

## How does it work?

- **Forward-only:** you can only move to the next row, never back.
- **Read-only:** you cannot change data through it.
- The reader starts **before** the first row. `Read()` moves to the next row and returns `true`, or `false` when there are no more rows.
- For an empty result, the first `Read()` returns `false`.

### Basic workflow (from Microsoft)

1. Run the command with `ExecuteReader()`.
2. Get the column ordinals (`GetOrdinal`).
3. Call `Read()` for each row.
4. Check nullable columns with `IsDBNull` before calling a typed getter.
5. **Dispose the reader** before reusing the connection or reading output parameters.

---

## Reading values: three ways

| Way | Example | Notes |
|---|---|---|
| By name | `dr["St_Fname"]` | Returns `object`. Easy, needs a cast |
| By ordinal | `dr[1]` | Returns `object`. Order must match the SELECT list |
| **Typed getter** | `dr.GetInt32(0)`, `dr.GetString(1)` | **Best performance**. No extra conversion/casting |

- `GetOrdinal("St_Id")` returns the column position. Call it **once** before the loop, not for every row.
- Typed getters do **not convert** types. `GetInt32` on a non-int column throws an exception. Use the getter that matches the SQL type.

---

## NULL values

| Code | When the value is NULL |
|---|---|
| `dr.GetString(i)` | Throws `SqlNullValueException` |
| `dr["St_Fname"].ToString()` | Returns `""` (hides the NULL) |
| `(int)dr["St_Age"]` | Throws (cannot cast `DBNull` to `int`) |
| `dr.IsDBNull(i)` | Returns `true`. **Check this first** |

---

## Example: read students into a list

```csharp
public class Student
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public int? Age { get; set; }
}
```

```csharp
var students = new List<Student>();

using (var con = new SqlConnection(cs))
using (var cmd = new SqlCommand("SELECT St_Id, St_Fname, St_Age FROM Student", con))
{
    con.Open();

    using (SqlDataReader dr = cmd.ExecuteReader())
    {
        int idOrd   = dr.GetOrdinal("St_Id");
        int nameOrd = dr.GetOrdinal("St_Fname");
        int ageOrd  = dr.GetOrdinal("St_Age");

        while (dr.Read())
        {
            students.Add(new Student
            {
                Id        = dr.GetInt32(idOrd),
                FirstName = dr.IsDBNull(nameOrd) ? null : dr.GetString(nameOrd),
                Age       = dr.IsDBNull(ageOrd) ? (int?)null : dr.GetInt32(ageOrd)
            });
        }
    }
}   // reader, command, and connection are all disposed

// Use `students` after the connection is closed.
```

---

## HasRows

`HasRows` tells you whether the current result has rows. You can usually just call `Read()` directly.

```csharp
if (dr.HasRows)
{
    while (dr.Read()) { /* ... */ }
}
else
{
    Console.WriteLine("No rows returned.");
}
```

---

## Multiple result sets: NextResult

One command (or stored procedure) can return several result sets. Finish the current one, then call `NextResult()`.

```csharp
using (SqlDataReader dr = cmd.ExecuteReader())
{
    do
    {
        while (dr.Read())
        {
            // read the current result set
        }
    }
    while (dr.NextResult());
}
```

Columns, names, ordinals, and types can change per result set. **Recompute cached ordinals** after `NextResult()` returns `true`.

---

## Important notes

- **The reader needs an open connection.** While it is open, the connection is busy serving it. You cannot run another command on it (unless MARS is enabled).
- **Return data, not the reader.** Copy rows into a `List<Student>` first. If the connection is disposed, the reader stops working.
- Keep the loop **short**. Slow work inside `while (dr.Read())` keeps the connection busy.
- **UI binding (Windows Forms):** bind a `DataGridView` to the **list** (or a `DataTable`), not to the reader.
- **Large values and binary data:** use `ExecuteReader(CommandBehavior.SequentialAccess)`. See `execute-methods.md`.
- If the command has **output parameters**, read them only **after** the reader is closed.

---

## Modern .NET differences

- Same concepts and same members (`Read`, `GetOrdinal`, typed getters, `IsDBNull`, `NextResult`).
- Use `Microsoft.Data.SqlClient` for new apps.
- Async methods exist (`ReadAsync`, `NextResultAsync`). Only one async operation can be pending on a reader at a time. See `asynchronous-ado-net.md`.

---

## Common mistakes

1. Reading values **before** calling `Read()`.
2. Calling `GetString` or an unchecked cast on a NULL value.
3. Using the wrong typed getter (for example, `GetInt32` on a `bigint` column).
4. Relying on ordinals with `SELECT *`. Name the columns in the query.
5. Returning the `SqlDataReader` from a method after disposing the connection.
6. Not disposing the reader (it keeps the connection busy).
7. Opening a second reader on the same connection (without MARS == Multiple Active Result Sets.).

---

## Summary

- `SqlDataReader` = fast, forward-only, read-only, one row at a time.
- Flow: `ExecuteReader()` -> `GetOrdinal` -> `while (Read())` -> typed getters -> dispose.
- Use typed getters and check `IsDBNull` for nullable columns.
- Copy data into objects. Do not pass the reader around.

---

## Next

[Stored Procedures](stored-procedures.md)