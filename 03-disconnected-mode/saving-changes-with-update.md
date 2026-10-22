# Saving Changes with Update

## Definition

`adapter.Update(dt)` sends the changes in the `DataTable` to the database.

## How does it work?

For each row, the adapter checks its `RowState`:

| RowState | Command used |
|---|---|
| `Added` | `InsertCommand` |
| `Modified` | `UpdateCommand` |
| `Deleted` | `DeleteCommand` |

You must give the adapter these commands. Each command uses parameters, and each parameter is linked to a column in the DataTable.

## Example

```csharp
using (SqlConnection con = new SqlConnection(connectionString))
{
    SqlDataAdapter adapter = new SqlDataAdapter(
        "SELECT St_Id, St_Fname FROM Student", con);

    // Insert
    adapter.InsertCommand = new SqlCommand(
        "INSERT INTO Student (St_Id, St_Fname) VALUES (@id, @name)", con);
    adapter.InsertCommand.Parameters.Add("@id", SqlDbType.Int, 0, "St_Id");
    adapter.InsertCommand.Parameters.Add("@name", SqlDbType.NVarChar, 50, "St_Fname");

    // Update
    adapter.UpdateCommand = new SqlCommand(
        "UPDATE Student SET St_Fname = @name WHERE St_Id = @id", con);
    adapter.UpdateCommand.Parameters.Add("@name", SqlDbType.NVarChar, 50, "St_Fname");
    adapter.UpdateCommand.Parameters.Add("@id", SqlDbType.Int, 0, "St_Id");

    // Delete
    adapter.DeleteCommand = new SqlCommand(
        "DELETE FROM Student WHERE St_Id = @id", con);
    adapter.DeleteCommand.Parameters.Add("@id", SqlDbType.Int, 0, "St_Id");

    // Load, change, save
    DataTable dt = new DataTable();
    adapter.Fill(dt);

    dt.Rows[0]["St_Fname"] = "Ahmed";   // this row becomes Modified

    int changedRows = adapter.Update(dt);
}
```

The last argument in `Parameters.Add` (like `"St_Fname"`) is the **source column**. It tells the adapter which column in the DataTable gives the value.

## Shortcut: SqlCommandBuilder

`SqlCommandBuilder` can create the Insert, Update, and Delete commands for you from the SELECT command.

```csharp
SqlCommandBuilder builder = new SqlCommandBuilder(adapter);
adapter.Update(dt);
```

It is easy, but it only works for a single table with a primary key. Writing the commands by hand gives you more control.

## Important notes

- If you do not set a command, `Update` fails for rows that need it. For example, a `Deleted` row needs a `DeleteCommand`.
- After a successful `Update`, the adapter calls `AcceptChanges()` and all rows go back to `Unchanged`.
- Another user may have changed the same row. Handle this case in real projects.