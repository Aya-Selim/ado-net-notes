# What is ADO.NET?

> **Scope:** .NET Framework ADO.NET. Differences in modern .NET are marked clearly.
> **Source:** [ADO.NET Overview](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/ado-net-overview)

---

## 1. Definition

ADO.NET is a set of classes that gives .NET code access to data sources. With it, an application can **connect** to a data source, **retrieve** data, **handle** it, and **update** it.

It provides consistent access to SQL Server, XML, and data sources exposed through OLE DB and ODBC.

It is the most **direct** way to access data in .NET Framework. Higher-level tools such as Entity Framework are built on the same ideas.

---

## 2. The Two Main Parts

ADO.NET separates **data access** from **data manipulation**. The two parts can be used separately or together.

| Part | Job | Main classes |
|---|---|---|
| **Data Provider** | Connects to the database, runs commands, returns results | `Connection`, `Command`, `DataReader`, `DataAdapter` |
| **DataSet** | Holds data **in memory**, disconnected from the database | `DataSet`, `DataTable`, `DataRow` |

A `DataSet` can also be used **without** a data provider, for example to manage local data or XML data.

```
Your C# code  <---->  Data Provider  <---->  Database
      ^
      |  (optional)
      v
   DataSet / DataTable   (in-memory data)
```

---

## 3. Core Objects of a Data Provider

([Official table](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/data-providers#core-objects-of-net-framework-data-providers))

| Object | Purpose | Base class | SQL Server class |
|---|---|---|---|
| `Connection` | Opens a connection to a data source | `DbConnection` | `SqlConnection` |
| `Command` | Runs a command, exposes `Parameters`, can run inside a `Transaction` | `DbCommand` | `SqlCommand` |
| `DataReader` | Reads a forward-only, read-only stream of data | `DbDataReader` | `SqlDataReader` |
| `DataAdapter` | Fills a `DataSet` and resolves updates with the data source | `DbDataAdapter` | `SqlDataAdapter` |

Other provider classes (covered later): `Transaction`, `CommandBuilder`, `ConnectionStringBuilder`, `Parameter`, `Exception`.

---

## 4. Data Providers in .NET Framework

| Provider | Namespace |
|---|---|
| SQL Server | `System.Data.SqlClient` |
| OLE DB | `System.Data.OleDb` |
| ODBC | `System.Data.Odbc` |
| Oracle | `System.Data.OracleClient` |
| EntityClient | `System.Data.EntityClient` |
| SQL Server Compact 4.0 | `System.Data.SqlServerCe` |

For SQL Server, use the SQL Server provider. It uses its own protocol, so there is no extra OLE DB or ODBC layer, and Microsoft recommends it over the OLE DB provider for SQL Server.

ADO.NET classes in .NET Framework are in `System.Data.dll`. The separately distributed SQL Server provider uses `Microsoft.Data.SqlClient.dll`.

---

## 5. Connected vs Disconnected

| | Connected | Disconnected |
|---|---|---|
| Main objects | `Connection` + `Command` + `DataReader` | `DataAdapter` + `DataSet` / `DataTable` |
| Data lives | Streamed from the database while reading | In memory (a local copy) |
| Style | Forward-only, read-only | Read, change, then send changes back |

Details: `02-connected-mode/` and `03-disconnected-mode/`.

---

## 6. Minimal Examples

### Connected: `SqlDataReader`

```csharp
using System;
using System.Data.SqlClient;

const string connectionString =
    "Server=.;Database=SchoolDb;Integrated Security=true;";

using (var connection = new SqlConnection(connectionString))
using (var command = new SqlCommand("SELECT Id, Name FROM Students", connection))
{
    connection.Open();

    using (SqlDataReader reader = command.ExecuteReader())
    {
        while (reader.Read())
            Console.WriteLine($"{reader.GetInt32(0)} - {reader.GetString(1)}");
    }
}
```

Flow: create connection -> create command -> `Open()` -> `ExecuteReader()` -> `Read()` in a loop -> dispose (`using`).

### Disconnected: `SqlDataAdapter` + `DataTable`

```csharp
using System;
using System.Data;
using System.Data.SqlClient;

const string connectionString =
    "Server=.;Database=SchoolDb;Integrated Security=true;";

var table = new DataTable();

using (var connection = new SqlConnection(connectionString))
using (var adapter = new SqlDataAdapter("SELECT Id, Name FROM Students", connection))
{
    adapter.Fill(table);
}

// Data is in memory and still usable after the connection is disposed.
foreach (DataRow row in table.Rows)
    Console.WriteLine($"{row["Id"]} - {row["Name"]}");
```

---

## 7. ADO.NET vs ADO and Entity Framework

- **ADO** is for native COM developers. **ADO.NET** gives similar functionality to managed code. Microsoft recommends ADO.NET, not ADO, for .NET applications.
- ADO.NET is the most direct data access method. For a higher-level model, Microsoft points to Entity Framework.

---

## 8. Modern .NET Differences

| Topic | .NET Framework | Modern .NET |
|---|---|---|
| SQL Server provider | `System.Data.SqlClient` | **`Microsoft.Data.SqlClient`** NuGet package. Microsoft says to use it for new SQL Server applications |
| Existing `System.Data.SqlClient` code | Works | See the [migration guide](https://learn.microsoft.com/en-us/sql/connect/ado-net/migrate-system-data-sql-client-to-microsoft-data-sql-client) |
| Core concepts (`Connection`, `Command`, `DataReader`, `DataAdapter`) | Same | Same |

Other provider differences will be checked in `04-comparison-and-advanced/framework-vs-modern-dotnet.md`.

---

## 9. Common Mistakes

1. Thinking ADO.NET means only `DataSet`. Many apps use only `Connection` + `Command` + `DataReader`.
2. Confusing `SqlConnection` (SQL Server class) with `DbConnection` (shared base class).
3. Mixing `System.Data.SqlClient` and `Microsoft.Data.SqlClient` in one project.
4. Not disposing connections, commands, and readers. Use `using`.

---

## 10. Summary

- ADO.NET = classes for data access in .NET.
- Two parts: **Data Provider** and **DataSet**.
- Four core provider objects: **Connection, Command, DataReader, DataAdapter**.
- Two styles: **Connected** and **Disconnected**.
- SQL Server: `System.Data.SqlClient` in .NET Framework, `Microsoft.Data.SqlClient` for new apps.

---

## Official Documentation

- [ADO.NET (home)](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/)
- [ADO.NET Overview](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/ado-net-overview)
- [.NET Framework Data Providers](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/data-providers)
- [ADO.NET Architecture](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/ado-net-architecture)
- [ADO.NET Code Examples](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/ado-net-code-examples)

---

## Next

[Connected Architecture](../02-connected-mode/connected-architecture.md)
