# Connection and Connection String

## Definition

`SqlConnection` represents a connection to a SQL Server database.
A **connection string** is the text that tells it how to connect.

## What is inside a connection string?

A connection string answers three questions:

| Question | Keyword | Example |
|---|---|---|
| Where is the server? | `Server` | `.` or `localhost` or `.\SQLEXPRESS` |
| Which database? | `Database` | `ITI` |
| Who are you? | `Trusted_Connection` or `User Id` + `Password` | `Trusted_Connection=True` |

Example (Windows authentication):

```csharp
string connectionString =
    "Server=.;Database=ITI;Trusted_Connection=True;TrustServerCertificate=True;";
```

> **Note:** `TrustServerCertificate=True` is only for local development.
> `Microsoft.Data.SqlClient` encrypts the connection by default, and a local server usually has no trusted certificate.

## Creating and opening a connection

```csharp
using Microsoft.Data.SqlClient;

using (SqlConnection con = new SqlConnection(connectionString))
{
    con.Open();
    // use the connection here
}   // the connection is closed here automatically
```

The `using` block closes the connection even if an error happens. This is safer than calling `con.Close()` by hand.

## Do not hard-code the connection string

Put it in a configuration file so you can change it without editing the code.

`appsettings.json`:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=.;Database=ITI;Trusted_Connection=True;TrustServerCertificate=True;"
  }
}
```

In Windows Forms with .NET Framework, the same idea is used with `App.config`.

> **Security tip:** Never push real passwords to GitHub. Use Windows authentication, or user secrets / environment variables.

## Next

[SqlCommand](sqlcommand.md)