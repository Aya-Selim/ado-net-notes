# Execute Methods

`SqlCommand` has four main ways to run a command. Choose the one that matches what you expect back.

| Method | Returns | Use it for |
|---|---|---|
| `ExecuteReader()` | `SqlDataReader` | SELECT that returns many rows |
| `ExecuteScalar()` | One value (`object`) | SELECT that returns one cell, like `COUNT(*)` |
| `ExecuteNonQuery()` | Number of affected rows (`int`) | INSERT, UPDATE, DELETE |
| `ExecuteXmlReader()` | `XmlReader` | Queries that end with `FOR XML` (SQL Server only) |

## ExecuteScalar

```csharp
SqlCommand cmd = new SqlCommand("SELECT COUNT(*) FROM Student", con);
con.Open();
int count = (int)cmd.ExecuteScalar();
```

If the query returns many rows or columns, only the **first cell** is returned.

## ExecuteNonQuery

```csharp
SqlCommand cmd = new SqlCommand(
    "UPDATE Student SET St_Fname = @name WHERE St_Id = @id", con);
cmd.Parameters.AddWithValue("@name", "Ahmed");
cmd.Parameters.AddWithValue("@id", 5);

con.Open();
int rows = cmd.ExecuteNonQuery();   // how many rows changed
```

## Note

For `SELECT`, `ExecuteNonQuery` returns `-1`. Do not use it to read data.

## Next

[Parameters and SQL Injection](parameters-and-sql-injection.md)