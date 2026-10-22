# DataTable and DataSet

## DataTable

A `DataTable` is one table stored in memory.

| Part | Class | Meaning |
|---|---|---|
| Columns | `DataColumn` | The structure (name and type) |
| Rows | `DataRow` | The data |

```csharp
DataTable dt = new DataTable();

// read a value from the first row
string name = dt.Rows[0]["St_Fname"].ToString();

// loop over all rows
foreach (DataRow row in dt.Rows)
{
    Console.WriteLine(row["St_Fname"]);
}
```

## RowState

Each `DataRow` remembers what happened to it since it was loaded. This is the `RowState` property.

| RowState | Meaning |
|---|---|
| `Unchanged` | Not changed |
| `Added` | New row, not in the database yet |
| `Modified` | A value was changed |
| `Deleted` | Marked for deletion |

The adapter uses `RowState` to decide which command to run when you call `Update()`.

## DataSet

A `DataSet` is a group of `DataTable` objects. You can also add relations between the tables.

```csharp
DataSet ds = new DataSet();
// the adapter can put more than one table in it
```

## DataTable or DataSet?

- Working with **one** table: use `DataTable`.
- Working with **many related** tables: use `DataSet`.

## Next

[SqlDataAdapter and Fill](sqldataadapter-fill.md)