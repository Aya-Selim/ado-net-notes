# .NET Framework Data Providers

> **Scope:** .NET Framework ADO.NET.
> **Source:** [.NET Framework Data Providers](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/data-providers)

---

## 1. What is a data provider?

A data provider is used to:

1. **Connect** to a database.
2. **Execute** commands.
3. **Retrieve** results.

Results are processed directly, placed in a `DataSet`, combined with other sources, or passed between tiers.

Providers are **lightweight**: a minimal layer between your code and the data source, so performance is good without losing functionality.

---

## 2. Providers included in .NET Framework

| Provider | Namespace | Data source |
|---|---|---|
| SQL Server | `System.Data.SqlClient` | Microsoft SQL Server |
| OLE DB | `System.Data.OleDb` | OLE DB data sources |
| ODBC | `System.Data.Odbc` | ODBC data sources |
| Oracle | `System.Data.OracleClient` | Oracle (needs Oracle client software 8.1.7+) |
| EntityClient | `System.Data.EntityClient` | Entity Data Model (EDM) apps |
| SQL Server Compact 4.0 | `System.Data.SqlServerCe` | SQL Server Compact 4.0 |

---

## 3. Classes every provider has

**Core objects** (same four in every provider):

| Object | Base class (`System.Data.Common`) |
|---|---|
| `Connection` | `DbConnection` |
| `Command` | `DbCommand` |
| `DataReader` | `DbDataReader` |
| `DataAdapter` | `DbDataAdapter` |

**Additional classes:**

| Object | Base class | Purpose |
|---|---|---|
| `Transaction` | `DbTransaction` | Enlists commands in a transaction |
| `CommandBuilder` | `DbCommandBuilder` | Generates `DataAdapter` commands, or fills `Parameters` from a stored procedure |
| `ConnectionStringBuilder` | `DbConnectionStringBuilder` | Builds and manages connection strings |
| `Parameter` | `DbParameter` | Input, output, and return-value parameters |
| `Exception` | `DbException` | Error from the **data source** (client errors throw a normal .NET exception) |
| `Error` | none listed | Details of a warning or error from the data source |

Each provider names these with its own prefix:

| Provider | Connection | Command | Reader | Adapter |
|---|---|---|---|---|
| SQL Server | `SqlConnection` | `SqlCommand` | `SqlDataReader` | `SqlDataAdapter` |
| OLE DB | `OleDbConnection` | `OleDbCommand` | `OleDbDataReader` | `OleDbDataAdapter` |
| ODBC | `OdbcConnection` | `OdbcCommand` | `OdbcDataReader` | `OdbcDataAdapter` |

---

## 4. SQL Server provider (SqlClient)

- Uses **its own protocol** to talk to SQL Server.
- Accesses SQL Server **directly**, with no OLE DB or ODBC layer. This makes it lightweight and fast.
- Supports **local** and **distributed** transactions. For distributed transactions it **enlists automatically** (using Windows Component Services or `System.Transactions`).

```
SqlClient : Your code -> SqlClient ---------------------> SQL Server
OLE DB    : Your code -> OleDb -> OLE DB Service -> OLE DB provider -> Data source
```

---

## 5. OLE DB, ODBC, and Oracle: key points

| Provider | Key points |
|---|---|
| **OLE DB** | Uses native OLE DB through **COM interop**. Does **not** support OLE DB 2.5 interfaces. Does **not** work with the OLE DB provider for ODBC (`MSDASQL`); use the ODBC provider instead. |
| **ODBC** | Uses the native ODBC **Driver Manager**. Requires MDAC 2.6 or later (2.8 SP1 recommended). |
| **Oracle** | Needs Oracle client software on the machine. Reference both `System.Data.dll` and `System.Data.OracleClient.dll`. |

---

## 6. Which provider should I choose?

| Provider | Recommendation from Microsoft |
|---|---|
| **SQL Server** | Recommended for middle-tier and single-tier apps using SQL Server. Recommended over OLE DB (`SQLOLEDB`) for SQL Server. |
| **OLE DB** | For SQL Server, use the SQL Server provider instead. Recommended for single-tier apps using **Microsoft Access**. Access as a middle-tier source is not recommended. |
| **ODBC** | Recommended for middle and single-tier apps using ODBC data sources. |
| **Oracle** | Recommended for middle and single-tier apps using Oracle data sources. |

**Rule of thumb:** use the provider made for your database. Use OLE DB or ODBC only when there is no dedicated provider.

---

## 7. EntityClient

- Used for data based on an **Entity Data Model**.
- Unlike the others, it does **not** talk to the data source directly. It uses **Entity SQL** and the underlying data provider.

---

## 8. Code example

### Include the provider namespace

```csharp
using System.Data.SqlClient;   // SQL Server
// using System.Data.OleDb;    // OLE DB
// using System.Data.Odbc;     // ODBC
```

### Use base classes to see the common design

```csharp
using System;
using System.Data.Common;
using System.Data.SqlClient;

const string connectionString =
    "Server=.;Database=SchoolDb;Integrated Security=true;";

using (DbConnection connection = new SqlConnection(connectionString))
using (DbCommand command = connection.CreateCommand())
{
    command.CommandText = "SELECT Id, Name FROM Students";
    connection.Open();

    using (DbDataReader reader = command.ExecuteReader())
    {
        while (reader.Read())
            Console.WriteLine($"{reader.GetInt32(0)} - {reader.GetString(1)}");
    }
}
```

Only `new SqlConnection(...)` is provider-specific. The rest uses the base classes. Full provider-independent code is covered later in `05-advanced-topics/provider-independent-programming.md`.

---

## 9. Modern .NET Differences

| Topic | .NET Framework | Modern .NET |
|---|---|---|
| SQL Server provider | `System.Data.SqlClient` | `Microsoft.Data.SqlClient` (recommended by Microsoft for new SQL Server apps) |
| Other providers (OLE DB, ODBC, Oracle, EntityClient, SQL Server Compact) | Listed in section 2 | **Not verified in this note.** Checked in `06-modern-dotnet-and-ecosystem/framework-vs-modern-dotnet.md` |

---

## 10. Common Mistakes

1. Using the OLE DB provider for SQL Server instead of `SqlClient`.
2. Trying to use the OLE DB provider with the ODBC OLE DB driver (`MSDASQL`).
3. Catching only `DbException` and forgetting that client-side errors throw normal .NET exceptions.
4. Using Access as the data source for a multithreaded app such as ASP.NET.

---

## 11. Summary

- A provider = connect, execute, retrieve.
- Four core objects: `Connection`, `Command`, `DataReader`, `DataAdapter`. Each has a base class in `System.Data.Common`.
- For SQL Server, use `SqlClient`. It is direct and has no OLE DB or ODBC layer.
- Use OLE DB for Access, ODBC for ODBC sources, and the Oracle provider for Oracle.

---

## Official Documentation

- [.NET Framework Data Providers](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/data-providers)
- [Transactions and Concurrency](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/transactions-and-concurrency)
- [System.Data.Common namespace](https://learn.microsoft.com/en-us/dotnet/api/system.data.common)

---

## Next

[Connected Architecture](../02-connected-mode/connected-architecture.md)