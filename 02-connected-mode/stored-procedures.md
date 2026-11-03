# Stored Procedures

> **Scope:** .NET Framework ADO.NET. Differences in modern .NET are marked.
> **Sources:** [Modify data with stored procedures (SqlClient)](https://learn.microsoft.com/en-us/sql/connect/ado-net/modify-data-with-stored-procedures), [Modifying Data with Stored Procedures (.NET Framework)](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/modifying-data-with-stored-procedures), [Configuring parameters](https://learn.microsoft.com/en-us/sql/connect/ado-net/configure-parameters)

---

## 1. What is it?

A stored procedure is SQL code saved **inside the database**. Your C# code calls it by name and passes parameters.

A procedure can send data back in **three ways**:

| Way | What it is | How you read it in C# |
|---|---|---|
| **Result set** | Rows from a `SELECT` inside the procedure | `ExecuteReader()` |
| **Output parameter** | A value assigned to an `OUTPUT` parameter | `parameter.Value` after execution |
| **Return value** | The **integer** from the T-SQL `RETURN` statement | A `ReturnValue` parameter |

---

## 2. Call a procedure

Rules:

1. `CommandType = CommandType.StoredProcedure`.
2. `CommandText` = the procedure name (schema-qualified, like `dbo.GetStudentsByAge`). **Only the name**.
3. Add **one parameter for each procedure parameter**, with matching **name, type, size, and direction**.

```sql
CREATE PROCEDURE dbo.GetStudentsByAge @MinAge INT
AS
BEGIN
    SELECT St_Id, St_Fname, St_Age FROM Student WHERE St_Age >= @MinAge;
END
```

```csharp
using System.Data;
using Microsoft.Data.SqlClient;   // or System.Data.SqlClient in .NET Framework

using (var con = new SqlConnection(cs))
using (var cmd = new SqlCommand("dbo.GetStudentsByAge", con))
{
    cmd.CommandType = CommandType.StoredProcedure;
    cmd.Parameters.Add("@MinAge", SqlDbType.Int).Value = 20;

    con.Open();
    using (SqlDataReader dr = cmd.ExecuteReader())
    {
        while (dr.Read())
            Console.WriteLine($"{dr.GetInt32(0)} - {dr.GetString(1)}");
    }
}
```

> Parameter names must **match** the procedure's parameter names (`@MinAge`). SqlClient does not support `?` placeholders.

You can also call a procedure as text (`EXEC dbo.GetStudentsByAge @MinAge`). Microsoft says using the `Parameters` collection with `CommandType.StoredProcedure` is better, because you can define parameters explicitly and read **output parameters** and **return values**.

---

## 3. Parameter directions

| `ParameterDirection` | Meaning |
|---|---|
| `Input` | Sends a value (**default**, no need to set) |
| `Output` | Receives a value from the procedure |
| `InputOutput` | Sends a value, then receives a new one |
| `ReturnValue` | Receives the integer from `RETURN` |

Rules from Microsoft:

- Set `Direction` for every parameter that is **not input-only**.
- Set `Size` for **variable-length** output and input/output parameters (for example, `NVarChar`). For these, `Size` is the **buffer size**, so the value can be **truncated** if `Size` is too small.
- An `InputOutput` parameter also needs an **initial `Value`**.
- A `ReturnValue` parameter is always an **integer**. It does **not** receive a result set or a value from a `SELECT`.

---

## 4. Output parameter

```sql
CREATE PROCEDURE dbo.InsertStudent
    @FirstName NVARCHAR(50),
    @NewId INT OUTPUT
AS
BEGIN
    INSERT INTO Student (St_Fname) VALUES (@FirstName);
    SET @NewId = SCOPE_IDENTITY();
END
```

```csharp
using (var con = new SqlConnection(cs))
using (var cmd = new SqlCommand("dbo.InsertStudent", con))
{
    cmd.CommandType = CommandType.StoredProcedure;
    cmd.Parameters.Add("@FirstName", SqlDbType.NVarChar, 50).Value = "Aya";

    SqlParameter newId = cmd.Parameters.Add("@NewId", SqlDbType.Int);
    newId.Direction = ParameterDirection.Output;

    con.Open();
    cmd.ExecuteNonQuery();

    int id = (int)newId.Value;   // read AFTER execution
}
```

`ExecuteNonQuery` returns no rows, but output parameters and return values **are still filled**.

---

## 5. Return value

```sql
CREATE PROCEDURE dbo.DeleteStudent @Id INT
AS
BEGIN
    DELETE FROM Student WHERE St_Id = @Id;
    RETURN @@ROWCOUNT;
END
```

```csharp
using (var con = new SqlConnection(cs))
using (var cmd = new SqlCommand("dbo.DeleteStudent", con))
{
    cmd.CommandType = CommandType.StoredProcedure;
    cmd.Parameters.Add("@Id", SqlDbType.Int).Value = 5;

    SqlParameter ret = cmd.Parameters.Add("@ReturnValue", SqlDbType.Int);
    ret.Direction = ParameterDirection.ReturnValue;

    con.Open();
    cmd.ExecuteNonQuery();

    int deleted = (int)ret.Value;
}
```

The return-parameter name is **your choice**. What matters is `Direction = ReturnValue`.

---

## 6. Output parameters with a reader

Output, input/output, and return values are **not final while the reader is open**. Dispose the reader first.

```csharp
SqlParameter total = cmd.Parameters.Add("@Total", SqlDbType.Int);
total.Direction = ParameterDirection.Output;

using (SqlDataReader dr = cmd.ExecuteReader())
{
    while (dr.Read())
    {
        // read rows
    }
}   // reader is disposed here

int count = (int)total.Value;   // now it is safe to read
```

---

## 7. SET NOCOUNT ON

- `SET NOCOUNT ON` is common inside procedures. It stops row counts from being reported.
- With it, `ExecuteNonQuery()` can return **`-1`** even when rows changed. Do not use its return value as the success check. Use an **output parameter** or **return value** instead.
- With `SqlDataAdapter` and a stored procedure for **update or delete**, do **not** use `SET NOCOUNT ON`. The adapter sees a row count of zero as a **concurrency conflict**, and a `DBConcurrencyException` is thrown (see `03-disconnected-mode/saving-changes-with-update.md`).

---

## 8. DeriveParameters (optional)

`SqlCommandBuilder.DeriveParameters` fills the `Parameters` collection from the procedure definition:

```csharp
cmd.CommandType = CommandType.StoredProcedure;
con.Open();
SqlCommandBuilder.DeriveParameters(cmd);   // one extra round trip to the server
```

It costs an **extra round trip** and overwrites existing parameters. If you know the parameters at design time, set them **explicitly**.

---

## 9. Security

A stored procedure is safe against SQL injection only if:

1. You pass input as **parameters** (not string concatenation).
2. The procedure itself does **not** build SQL text by joining strings.

See `parameters-and-sql-injection.md`.

---

## 10. Modern .NET differences

- Same pattern: `CommandType.StoredProcedure`, same directions, same rules.
- Use `Microsoft.Data.SqlClient` for new apps.

---

## 11. Common mistakes

1. Forgetting `CommandType.StoredProcedure`.
2. Wrong parameter name (it must match the procedure).
3. Not setting `Direction` for output or return-value parameters.
4. Not setting `Size` for `NVarChar`/`VarChar` output parameters (the value can be truncated).
5. Reading output parameters **before** the reader is closed.
6. Expecting `RETURN` to give a result set. It gives one **integer**.
7. Trusting `ExecuteNonQuery()` when the procedure uses `SET NOCOUNT ON`.
8. Passing C# `null` instead of `DBNull.Value`.
9. Building dynamic SQL by concatenation inside the procedure.

---

## 12. Summary

- Call by name: `CommandType.StoredProcedure` + parameters that match.
- Three ways back: **result set** (reader), **output parameter**, **return value** (integer).
- Set `Direction`, and `Size` for variable-length output parameters.
- Read output and return values **after** execution and after the reader is closed.
- `SET NOCOUNT ON` makes `ExecuteNonQuery()` unreliable. Use output or return values.

---

## Next

[Null and Data Type Mapping](null-and-data-type-mapping.md)