# SqlCommand

## Definition

`SqlCommand` holds the SQL statement (or stored procedure name) that you want to run, and the connection to run it on.

## Main properties

| Property | Meaning |
|---|---|
| `CommandText` | The SQL query or the stored procedure name |
| `CommandType` | `Text` (default), `StoredProcedure`, or `TableDirect` |
| `Connection` | The `SqlConnection` object to use |
| `Parameters` | Values you send to the query. See [Parameters](parameters-and-sql-injection.md) |

## Example

Long way:

```csharp
SqlCommand cmd = new SqlCommand();
cmd.CommandText = "SELECT St_Id, St_Fname FROM Student";
cmd.CommandType = CommandType.Text;   // this is the default
cmd.Connection = con;
```

Short way (most common):

```csharp
SqlCommand cmd = new SqlCommand("SELECT St_Id, St_Fname FROM Student", con);
```

## Running a stored procedure

```csharp
SqlCommand cmd = new SqlCommand("GetAllStudents", con);
cmd.CommandType = CommandType.StoredProcedure;
```

## Common mistake

Forgetting to set the `Connection` property. You will get an error when you execute the command.

## Next

[SqlDataReader](sqldatareader.md)