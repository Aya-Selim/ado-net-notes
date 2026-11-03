# Resource Management and `using`

> **Scope:** .NET Framework ADO.NET. Differences in modern .NET are marked.
> **Sources:** [SqlConnection.Close](https://learn.microsoft.com/en-us/dotnet/api/system.data.sqlclient.sqlconnection.close), [Retrieving Data Using a DataReader](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/retrieving-data-using-a-datareader), [using statement (C#)](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/statements/using)

---

## 1. The problem

A database connection is a **limited resource**. If you do not release it, the connection stays busy and your app (or the server) can run out of connections.

The key fact from Microsoft:

> If a `SqlConnection` goes out of scope, it is **not closed**. You must close it explicitly with `Close()` or `Dispose()`.

The garbage collector will **not** save you here.

---

## 2. `IDisposable` and `using`

`SqlConnection`, `SqlCommand`, and `SqlDataReader` are all **disposable** (`IDisposable`). They hold resources that must be released.

A `using` block calls `Dispose()` for you when the block ends, **even if an exception happens**.

```csharp
using (var connection = new SqlConnection(cs))
{
    connection.Open();
    // work
}   // Dispose() is called here
```

It is the same as writing `try/finally` by hand:

```csharp
var connection = new SqlConnection(cs);
try
{
    connection.Open();
    // work
}
finally
{
    connection.Dispose();   // always runs
}
```

---

## 3. `Close()` vs `Dispose()`

| Fact (from Microsoft docs) | Meaning |
|---|---|
| `Close` and `Dispose` are **functionally equivalent** for `SqlConnection` | Either one releases the connection |
| You can call `Close` more than once | No exception |
| Pooling **on** (default) | The connection returns to the **connection pool** (not truly closed) |
| Pooling **off** | The connection to the server is really closed |
| `Close` rolls back pending transactions | Uncommitted work is lost |

Pooling is explained in `connection-pooling.md`.

---

## 4. What to dispose, and the standard pattern

Dispose **all three**: connection, command, and reader. Nest the `using` blocks:

```csharp
using (var connection = new SqlConnection(cs))
using (var command = new SqlCommand("SELECT Id, Name FROM Students", connection))
{
    connection.Open();

    using (SqlDataReader reader = command.ExecuteReader())
    {
        while (reader.Read())
            Console.WriteLine($"{reader.GetInt32(0)} - {reader.GetString(1)}");
    }   // reader disposed first
}       // then command, then connection
```

Objects are disposed in **reverse order** of creation.

---

## 5. DataReader rules (from Microsoft docs)

- **Always close the reader** when you finish with it.
- While a reader is open, the connection is **in use exclusively** by that reader. You cannot run another command on it (including another reader).
- If a command has **output parameters or return values**, they are **not available until the reader is closed**.

---

## 6. Exceptions do not close the connection for you

If a `SqlException` happens while running a command, the connection **stays open** when the error severity is **19 or less**. So you still need `using` (or `finally`) to release it.

---

## 7. Modern C# syntax: `using` declaration

C# 8 and later allow a shorter form. The object is disposed at the end of the **enclosing scope** (usually the method).

```csharp
using var connection = new SqlConnection(cs);
using var command = new SqlCommand("SELECT COUNT(*) FROM Students", connection);

connection.Open();
int count = (int)command.ExecuteScalar();
```

Same behavior, less nesting. In .NET Framework projects the default C# version is older, so the classic `using` block is the safe choice in these notes.

---

## 8. Modern .NET differences

| Topic | .NET Framework | Modern .NET |
|---|---|---|
| `IDisposable` and `using` rules | Same | Same |
| `using` declaration (C# 8) | Not the default language version | Available with modern C# |
| Package | `System.Data.SqlClient` | `Microsoft.Data.SqlClient` (same `Close`/`Dispose` behavior documented) |

---

## 9. Common mistakes

1. **No `using`**, or calling `Close()` only at the end of the code (it is skipped when an exception happens).
2. **Returning a `SqlDataReader` from a method** whose `using` already disposed the connection. The reader becomes unusable. Return the **data** (for example, a `List<Student>`) instead.
3. **Disposing only the connection** and forgetting the reader.
4. **Opening the connection early** and doing slow work before using it. Open late, close early.
5. **Calling `Close` or `Dispose` on connections or readers inside a finalizer** of your own class. Microsoft says not to.
6. **Sharing one static connection** across the app. Create a connection per operation and let pooling handle reuse.

---

## 10. Summary

- Not disposed = not closed. Going out of scope does **not** close a connection.
- Use `using` for `SqlConnection`, `SqlCommand`, and `SqlDataReader`.
- `Close` and `Dispose` are equivalent. With pooling, the connection returns to the pool.
- Always close the reader. An open reader blocks the connection.
- Return data, not readers.

---

## Next

[Connection Pooling](connection-pooling.md)