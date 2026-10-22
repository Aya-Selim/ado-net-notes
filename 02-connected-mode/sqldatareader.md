# SqlDataReader

## Definition

`SqlDataReader` reads the rows returned by a SELECT query, one row at a time.

## Why do we use it?

It is very fast and uses little memory, because it does not load all rows at once.

## How does it work?

- **Forward-only:** you can only move to the next row, never back.
- **Read-only:** you cannot change data through it.
- `Read()` moves to the next row. It returns `true` if there is a row, and `false` when there are no more rows.
- At the start, the reader is **before** the first row, so you must call `Read()` once before reading.

## Example: Read students into a list

```csharp
public class Student
{
    public int Id { get; set; }
    public string FirstName { get; set; }
}
```

```csharp
List<Student> students = new List<Student>();

using (SqlConnection con = new SqlConnection(connectionString))
using (SqlCommand cmd = new SqlCommand("SELECT St_Id, St_Fname FROM Student", con))
{
    con.Open();

    using (SqlDataReader dr = cmd.ExecuteReader())
    {
        while (dr.Read())
        {
            Student s = new Student();
            s.Id = (int)dr["St_Id"];
            s.FirstName = dr["St_Fname"].ToString();
            students.Add(s);
        }
    }
}

dgv_student.DataSource = students;   // Windows Forms DataGridView
```

Use the **same column names** as in the database.

## Important notes

- **The reader needs an open connection.** If the connection closes, the reader stops working. That is why we copy the data into a `List<Student>` first, and use it after.
- **Windows Forms:** you cannot bind a `DataGridView` directly to a `SqlDataReader`. Copy the data into a list first, as in the example.
- **Web Forms:** a `GridView` can use a reader directly as its data source.
- **NULL values:** if a column can be NULL, `dr["St_Fname"].ToString()` returns an empty string, but `(int)dr["Age"]` will crash. Check with `dr.IsDBNull(index)` first.

## Next

[Execute Methods](execute-methods.md)