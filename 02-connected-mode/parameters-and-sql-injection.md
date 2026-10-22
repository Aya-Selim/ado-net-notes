# Parameters and SQL Injection

## What is SQL injection?

SQL injection happens when a user types SQL code into an input box, and your app runs it as part of the query.

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

## The solution: parameters

A parameter sends the value **separately** from the SQL text. The database treats it as data only, never as code.

```csharp
string query = "SELECT St_Id, St_Fname FROM Student WHERE St_Fname = @name";

using (SqlCommand cmd = new SqlCommand(query, con))
{
    cmd.Parameters.Add("@name", SqlDbType.NVarChar, 50).Value = txtName.Text;

    con.Open();
    using (SqlDataReader dr = cmd.ExecuteReader())
    {
        // read the rows
    }
}
```

## Good to know

- `AddWithValue` is shorter, but SQL Server has to guess the data type, which can cause slow queries. `Parameters.Add` with an explicit type is the better habit.
- Parameters protect **values** only. You cannot use them for table or column names.
- Stored procedures with parameters are also safe. But a stored procedure that builds SQL text by joining strings is **not** safe.

## Rule

Never build a SQL query by joining user input into a string. Always use parameters.