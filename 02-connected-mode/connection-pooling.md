# Connection Pooling

> **Scope:** .NET Framework ADO.NET. Differences in modern .NET are marked.
> **Source:** [SQL Server Connection Pooling (ADO.NET)](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/sql-server-connection-pooling)

---

## 1. The problem

Opening a database connection is **expensive**. It needs several steps: open a network channel (socket or named pipe), do the handshake with the server, parse the connection string, authenticate, and more.

Most apps open and close the **same kind of connection** again and again. Doing all of that each time is wasteful.

---

## 2. The solution: connection pooling

ADO.NET keeps a **pool** of already-open physical connections and reuses them.

- It is **enabled by default**. You do not write any code for it.
- `Open()` takes a connection from the pool (or creates one if needed).
- `Close()` / `Dispose()` **returns** it to the pool. It is not really closed.

```
Open()   --->  [ POOL ]  --->  free connection given to you
                  ^
Close()  ---------+   connection goes back to the pool, ready for reuse
```

Result: better **performance and scalability**.

---

## 3. One pool per exact connection string

A pool is created for each unique connection string, using an **exact match**.

- Different text = different pool. Even **keywords in a different order** create a separate pool.
- Pools are separate per **process**, per **application domain**, per connection string, and (with integrated security) per **Windows identity**.

```csharp
using (var c1 = new SqlConnection("Server=.;Database=ITI;Integrated Security=true;"))
{
    c1.Open();   // creates pool A
}

using (var c2 = new SqlConnection("Server=.;Database=Shop;Integrated Security=true;"))
{
    c2.Open();   // different string -> creates pool B
}

using (var c3 = new SqlConnection("Server=.;Database=ITI;Integrated Security=true;"))
{
    c3.Open();   // same as c1 -> reuses pool A
}
```

> **Tip:** Build one connection string and reuse it everywhere. Do not put per-request values in it.

---

## 4. Pool settings (connection string keywords)

| Keyword | Default | Meaning |
|---|---|---|
| `Pooling` | `true` | Use pooling |
| `Max Pool Size` | `100` | Maximum connections in the pool |
| `Min Pool Size` | `0` | Minimum connections kept in the pool |

```csharp
string cs = "Server=.;Database=ITI;Integrated Security=true;Max Pool Size=50;";
```

---

## 5. Life of a pooled connection

| Event | What happens |
|---|---|
| First `Open()` | Pool is created (plus connections to meet `Min Pool Size`) |
| More demand | New connections are added, up to `Max Pool Size` |
| Pool is full and all connections are busy | New `Open()` calls **wait**. If the wait is longer than `Connect Timeout`, `Open()` fails |
| Connection idle ~4 to 8 minutes | Removed from the pool |
| Server connection is broken | Removed from the pool |
| Fatal error (for example, failover) | The pool is cleared |

- If `Min Pool Size` is 0 (default), idle pools are closed after inactivity.
- A broken connection is detected only when you **try to use it**. The first command on it can throw an exception.
- Pending transactions are rolled back when the connection is reset.

### Clear pools manually (rarely needed)

```csharp
SqlConnection.ClearAllPools();      // all pools of this provider
SqlConnection.ClearPool(connection); // only the pool of this connection
```

---

## 6. Pool exhaustion (the classic bug)

If you open connections and **never close them**, they never return to the pool. When the pool reaches `Max Pool Size`, new `Open()` calls wait and then fail.

```csharp
// BAD: connections are never returned to the pool
for (int i = 0; i < 200; i++)
{
    var con = new SqlConnection(cs);
    con.Open();   // fails once the pool is exhausted
}

// GOOD: each connection returns to the pool after the block
for (int i = 0; i < 200; i++)
{
    using (var con = new SqlConnection(cs))
    {
        con.Open();
    }
}
```

This is why `using` matters (see `resource-management-and-using.md`).

---

## 7. Modern .NET differences

| Topic | .NET Framework | Modern .NET |
|---|---|---|
| Concept (one pool per exact string) | Yes | Same |
| Keywords `Pooling`, `Min Pool Size`, `Max Pool Size` | Yes | Same, with the same defaults |
| Package | `System.Data.SqlClient` | `Microsoft.Data.SqlClient` |

Newer `Microsoft.Data.SqlClient` versions add more pooling-related keywords. Check the docs for your version before using them.

---

## 8. Common mistakes

1. **Not closing connections** (pool exhaustion).
2. **Many slightly different connection strings** (many pools). Reuse one string.
3. **Keeping one static connection open for the whole app** instead of opening per operation and letting the pool reuse.
4. **Setting `Pooling=false`** in production without a reason.
5. **Raising `Max Pool Size`** to hide a leak. Fix the leak first.
6. **Thinking `Close()` closes the network connection.** With pooling on, it only returns it to the pool.

---

## 9. Summary

- Pooling reuses open connections because opening is expensive. It is on by default.
- `Open()` takes from the pool. `Close()`/`Dispose()` returns to it.
- One pool per exact connection string. Defaults: `Max Pool Size=100`, `Min Pool Size=0`.
- Not closing connections exhausts the pool. Always use `using`.

---

## Next

[SqlCommand](sqlcommand.md)