# Parameters and SQL Injection

> **Scope:** .NET Framework ADO.NET. Differences in modern .NET are marked.
> **Sources:** [Configuring parameters and parameter data types](https://learn.microsoft.com/en-us/sql/connect/ado-net/configure-parameters), [SQL injection](https://learn.microsoft.com/sql/relational-databases/security/sql-injection)

---

## 1. What is SQL injection?

SQL injection happens when a user types SQL code into an input, and your app runs it as part of the query. Microsoft says string concatenation is the **primary point of entry** for it.

### Bad code (never do this)

```csharp
string name = txtName.Text;
string query = "SELECT * FROM Student WHERE St_Fname = '" + name + "'";
```

If the user types:

```
' OR '1'='1
```

The query becomes:

```sql
SELECT * FROM Student WHERE St_Fname = '' OR '1'='1'
```

This returns **all** students. An attacker can do much worse, like deleting data.

---

## 2. The solution: parameters

A parameter sends the value to the command **separately** from the SQL text. Microsoft says parameter input is treated as a **literal value, not executable code**.

```csharp
string query = "SELECT St_Id, St_Fname FROM Student WHERE St_Fname = @name";

using (var con = new SqlConnection(cs))
using (var cmd = new SqlCommand(query, con))
{
    cmd.Parameters.Add("@name", SqlDbType.NVarChar, 50).Value = txtName.Text;

    con.Open();
    using (SqlDataReader dr = cmd.ExecuteReader())
    {
        // read the rows
    }
}
```

More benefits (from Microsoft docs):

- **Type checking and length validation**. If the value does not match the type or length, an exception is thrown.
- Text commands with parameters run through `sp_executesql`, so SQL Server can **reuse the execution plan**.

---

## 3. Parameter rules for SqlClient

| Rule | Detail |
|---|---|
| Names start with `@` | `@name`, `@id` |
| No `?` placeholders | `?` is for OLE DB and ODBC, not SqlClient |
| Parameters are for **values** only | Not for table names, column names, or SQL keywords |
| Stored procedure parameters | Names must match the procedure's parameter names (for example, `@CustomerID`) |

---

## 4. Add, AddWithValue, and the type

```csharp
// Recommended: explicit type (and size for strings)
cmd.Parameters.Add("@name", SqlDbType.NVarChar, 50).Value = txtName.Text;
cmd.Parameters.Add("@id", SqlDbType.Int).Value = 5;

// Shorter: type is inferred from the .NET value
cmd.Parameters.AddWithValue("@name", txtName.Text);
```

- `AddWithValue` makes ADO.NET **infer** the type from the .NET value. It is fine when the inferred type is stable and matches the database column.
- Microsoft says to **prefer `Add` with an explicit `SqlDbType`** for production commands when the schema is known.
- A mismatch between the inferred type and the column type can cause implicit conversions, which may slow the query.

---

## 5. NULL values

Send `DBNull.Value`, **not** C# `null`.

```csharp
int? age = null;
cmd.Parameters.Add("@age", SqlDbType.Int).Value = (object)age ?? DBNull.Value;
```

Microsoft also says: do not pass C# `null` to `AddWithValue`. Specify the type and assign `DBNull.Value`.

---

## 6. Full example: UPDATE with several parameters

```csharp
string sql = "UPDATE Student SET St_Fname = @name, St_Age = @age WHERE St_Id = @id";

using (var con = new SqlConnection(cs))
using (var cmd = new SqlCommand(sql, con))
{
    cmd.Parameters.Add("@name", SqlDbType.NVarChar, 50).Value = txtName.Text;
    cmd.Parameters.Add("@age", SqlDbType.Int).Value = age;
    cmd.Parameters.Add("@id", SqlDbType.Int).Value = id;

    con.Open();
    int rows = cmd.ExecuteNonQuery();
}
```

---

## 7. Parameter direction (preview)

| `ParameterDirection` | Meaning |
|---|---|
| `Input` | Sends a value. **Default** |
| `Output` | Receives a value back |
| `InputOutput` | Both |
| `ReturnValue` | A stored procedure's return value |

Used mostly with stored procedures. See `stored-procedures.md`.

---

## 8. Good to know

- **Stored procedures are safe only if** you pass input as parameters **and** the procedure does not build SQL text from strings. Dynamic SQL inside a procedure can still be injectable.
- Microsoft also warns that even parameterized data can be manipulated by a determined attacker. Also **validate input** (type, length, format, range). Filtering characters alone is not a reliable defense.
- **LIKE:** parameters do not escape wildcards. If the user types `%` or `_`, they still act as wildcards.

```csharp
cmd.Parameters.Add("@pattern", SqlDbType.NVarChar, 50).Value = "%" + name + "%";
```

- **Dynamic table or column names:** a parameter cannot do this. Check the name against a fixed list of allowed names in your code.

---

## 9. Modern .NET differences

- Same rules: named `@parameters`, no `?` placeholders, same `Add` / `AddWithValue`.
- Use `Microsoft.Data.SqlClient` for new apps.
- The same idea applies to EF Core: do not concatenate user input into raw SQL.

---

## 10. Common mistakes

1. Concatenating or interpolating user input (`$"... {name}"`) into SQL.
2. Using `?` placeholders with SqlClient.
3. Passing C# `null` instead of `DBNull.Value`.
4. Trying to parameterize a table or column name.
5. Using `AddWithValue` everywhere without checking the type.
6. Forgetting that LIKE wildcards in the input are still active.
7. Building dynamic SQL by string concatenation inside a stored procedure.

---

## 11. Summary

- Never join user input into SQL text. Always use parameters.
- Use `Add` with `SqlDbType` (and size). `AddWithValue` is a shortcut that infers the type.
- Use `DBNull.Value` for NULL.
- Parameters protect **values**, not identifiers. Also validate input.

---

## Next

[SqlDataReader](sqldatareader.md)