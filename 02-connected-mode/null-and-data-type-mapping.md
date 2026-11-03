# NULL and Data Type Mapping

> **Scope:** .NET Framework ADO.NET. Differences in modern .NET are marked.
> **Sources:** [SQL Server Data Type Mappings](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/sql-server-data-type-mappings), [Handling Null Values](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/sql/handling-null-values)

---

## 1. Why mapping matters

SQL Server and .NET have **different type systems**. Example: .NET `Decimal` has a maximum scale of **28**, but SQL Server `decimal` has **38**.

So you must use the right .NET type (and the right getter) for each SQL column.

---

## 2. Common mappings

| SQL Server type | .NET type | `SqlDataReader` getter |
|---|---|---|
| `bigint` | `Int64` | `GetInt64` |
| `int` | `Int32` | `GetInt32` |
| `smallint` | `Int16` | `GetInt16` |
| `tinyint` | `Byte` | `GetByte` |
| `bit` | `Boolean` | `GetBoolean` |
| `decimal`, `numeric`, `money`, `smallmoney` | `Decimal` | `GetDecimal` |
| `float` | `Double` | `GetDouble` |
| `real` | `Single` | `GetFloat` |
| `char`, `varchar`, `nchar`, `nvarchar`, `text` | `String` | `GetString` |
| `date`, `datetime`, `datetime2`, `smalldatetime` | `DateTime` | `GetDateTime` |
| `datetimeoffset` | `DateTimeOffset` | `GetDateTimeOffset` |
| `time` | `TimeSpan` | `GetTimeSpan` |
| `uniqueidentifier` | `Guid` | `GetGuid` |
| `binary`, `varbinary`, `image`, `rowversion` | `Byte[]` | `GetBytes` |

- `SqlDbType` names usually match the SQL type: `SqlDbType.Int`, `SqlDbType.NVarChar`, `SqlDbType.BigInt`.
- The reader also has `GetSqlXxx` getters (for example, `GetSqlInt32`). They return `System.Data.SqlTypes` objects, to keep data integrity.

---

## 3. What is NULL?

In a database, NULL means the value is **unknown or missing**. It is **not** an empty string and **not** zero.

In ADO.NET, a database NULL is represented by **`DBNull.Value`** (not C# `null`).

| Situation | What you get / must do |
|---|---|
| Read a NULL column with `dr["col"]` | `DBNull.Value` |
| Read a NULL column with `dr.GetString(i)` | Throws `SqlNullValueException` |
| `ExecuteScalar()`, first cell is NULL | `DBNull.Value` |
| `ExecuteScalar()`, no rows | `null` |
| Send NULL as a parameter | Use `DBNull.Value`, **not** `null` |

---

## 4. Code

### Read a nullable column

```csharp
int ageOrd = dr.GetOrdinal("St_Age");

int? age = dr.IsDBNull(ageOrd) ? (int?)null : dr.GetInt32(ageOrd);
```

### Send a nullable value

```csharp
int? age = null;

cmd.Parameters.Add("@age", SqlDbType.Int).Value = (object)age ?? DBNull.Value;
```

### SqlTypes (optional)

Types in `System.Data.SqlTypes` (like `SqlInt32`) have an `IsNull` property and can hold NULL without an exception:

```csharp
SqlInt32 id = dr.GetSqlInt32(0);
bool isNull = id.IsNull;
```

### DataRow (preview of disconnected mode)

```csharp
bool isNull = row.IsNull("St_Age");   // true for a NULL value
```

---

## 5. Modern .NET differences

- Same mapping and same NULL rules.
- Use `Microsoft.Data.SqlClient` for new apps.
- Microsoft's current NULL page: [Handle null values (SqlClient)](https://learn.microsoft.com/en-us/sql/connect/ado-net/sql/handle-null-values).

---

## 6. Common mistakes

1. Using `(int)dr["col"]` or `GetString` on a column that can be NULL.
2. Passing C# `null` instead of `DBNull.Value` as a parameter.
3. Using the wrong getter (for example, `GetInt32` on a `bigint` column).
4. Treating NULL as an empty string or zero.
5. Casting `ExecuteScalar()` without checking `null` / `DBNull.Value`.

---

## 7. Summary

- Match the .NET type and getter to the SQL type (use the table above).
- Database NULL = `DBNull.Value`. Check with `IsDBNull` before reading.
- Send NULL with `DBNull.Value`. Use `int?` and `??` to convert easily.

---

## Next

[Exception Handling](../03-transactions-and-reliability/exception-handling.md)