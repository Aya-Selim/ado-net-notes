# SqlDataAdapter and Fill

## Definition

`SqlDataAdapter` moves data between the database and a `DataTable` or `DataSet`.

## Loading data with Fill

```csharp
using Microsoft.Data.SqlClient;
using System.Data;

DataTable dt = new DataTable();

using (SqlConnection con = new SqlConnection(connectionString))
{
    SqlDataAdapter adapter = new SqlDataAdapter(
        "SELECT St_Id, St_Fname FROM Student", con);

    adapter.Fill(dt);
}

dgv_student.DataSource = dt;
```

## What does Fill do?

1. Opens the connection (if it is closed).
2. Runs the SELECT command.
3. Copies the rows into the `DataTable`.
4. Closes the connection (if the adapter opened it).

You do **not** need to call `Open()` or `Close()` yourself.

## Important notes

- `Fill` only **loads** data. It does not save anything to the database.
- If you open the connection yourself before `Fill`, the adapter leaves it open. You must close it.
- A `DataTable` can be bound directly to a `DataGridView`.

## Next

[Saving Changes with Update](saving-changes-with-update.md)