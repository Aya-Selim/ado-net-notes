# ADO.NET Architecture

> **Scope:** .NET Framework ADO.NET.
> **Source:** [ADO.NET Architecture](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/ado-net-architecture)

---

## 1. Why this architecture?

- Data processing was traditionally **connection-based** and **two-tier**.
- Modern apps use **multi-tier** architectures, so programmers moved to a **disconnected** approach for better **scalability**.

ADO.NET supports both styles.

---

## 2. The Two Main Components

| Component | Designed for |
|---|---|
| **.NET Framework Data Provider** | Data manipulation and **fast, forward-only, read-only** access |
| **DataSet** | Data access **independent of any data source** |

```
 Data Source <---> Data Provider <---> DataSet
                  (Connection,         (DataTables,
                   Command,             relations,
                   DataReader,          constraints)
                   DataAdapter)
```

---

## 3. Data Provider Objects

| Object | Role |
|---|---|
| `Connection` | Connects to the data source |
| `Command` | Returns data, modifies data, runs stored procedures, sends or gets parameter info |
| `DataReader` | High-performance stream of data from the data source |
| `DataAdapter` | **Bridge** between the `DataSet` and the data source |

How they work together:

- `Command` uses a `Connection` to run SQL.
- `DataAdapter` uses `Command` objects to:
  1. **Load** the `DataSet` from the data source.
  2. **Send changes** made in the `DataSet` back to the data source.
- `DataAdapter` uses a `DataReader` internally to fill a `DataSet`.

---

## 4. The DataSet

- Works with different data sources, XML data, or data local to the application.
- Contains one or more **`DataTable`** objects (rows and columns).
- Also stores **primary key, foreign key, constraint, and relation** information.

---

## 5. DataReader or DataSet?

Use a **`DataSet`** to:

- Cache data locally and manipulate it.
- Pass data between tiers or from an XML Web service.
- Interact with data dynamically (for example, UI binding, or combining data from several sources).
- Do heavy processing **without an open connection**, which frees the connection for other clients.

Use a **`DataReader`** if you only need to **read** the results. It is faster and uses less memory because it avoids creating and filling a `DataSet`.

| | DataReader | DataSet |
|---|---|---|
| Access | Forward-only, read-only | Read and write in memory |
| Memory | Low | Higher |
| Connection | Open while reading | Not needed after filling |
| Best for | Simple reads | Caching, editing, relating data |

---

## 6. XML and ADO.NET

- ADO.NET was designed together with the .NET XML classes.
- A `DataSet` can be filled from XML (file or stream).
- A `DataSet` can be written as XML, including its schema as XSD.
- Because its native serialization format is XML, a `DataSet` is good for moving data between tiers.

---

## 7. Related Technologies (names only)

| Technology | One-line description |
|---|---|
| LINQ to DataSet | LINQ queries over data cached in a `DataSet` |
| LINQ to SQL | Queries over an object model mapped directly to database tables |
| ADO.NET Entity Framework | Program against a conceptual model instead of the storage schema |
| WCF Data Services | Deploy data services over HTTP on the Web or an intranet |

---

## 8. Modern .NET Differences

- The core design (data provider + `DataSet`) is the same idea.
- For SQL Server, new apps should use `Microsoft.Data.SqlClient` (see `what-is-ado-net.md`).
- Availability of the related technologies in section 7 will be checked in `06-modern-dotnet-and-ecosystem/framework-vs-modern-dotnet.md`.

---

## 9. Common Mistakes

1. Using a `DataSet` when you only need to read data. Use `DataReader`.
2. Thinking a `DataSet` is a live view of the database. It is a **local copy**.
3. Forgetting that `DataAdapter` is the part that connects the provider side to the `DataSet`.

---

## 10. Summary

- Two components: **Data Provider** and **DataSet**.
- `DataAdapter` is the bridge. It uses `Command` objects to load and save.
- `DataReader` = fast, read-only. `DataSet` = cached, editable, disconnected.

---

## Official Documentation

- [ADO.NET Architecture](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/ado-net-architecture)
- [Retrieving Data Using a DataReader](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/retrieving-data-using-a-datareader)
- [Populating a DataSet from a DataAdapter](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/populating-a-dataset-from-a-dataadapter)
- [DataSets, DataTables, and DataViews](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/dataset-datatable-dataview/)

---

## Next

[Data Providers](data-providers.md)