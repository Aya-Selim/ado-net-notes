# Connection and Connection String

> **Scope:** .NET Framework ADO.NET. Examples use `Microsoft.Data.SqlClient`, which Microsoft recommends for new SQL Server apps. For `System.Data.SqlClient`, only the `using` namespace changes.
> **Sources:** [Connection string syntax](https://learn.microsoft.com/en-us/sql/connect/ado-net/connection-string-syntax), [Encryption and certificate validation](https://learn.microsoft.com/en-us/sql/connect/ado-net/encryption-and-certificate-validation)

---

## Definition

`SqlConnection` represents a connection to a SQL Server database.
A **connection string** is the text that tells it how to connect.

---

## What is inside a connection string?

| Question | Keyword | Example |
|---|---|---|
| Where is the server? | `Server` (or `Data Source`) | `.`, `localhost`, `.\SQLEXPRESS` |
| Which database? | `Database` (or `Initial Catalog`) | `ITI` |
| Who are you? | `Integrated Security` or `User ID` + `Password` | `Integrated Security=true` |

Keywords are not case-sensitive.

### Windows authentication (preferred)

```csharp
string cs = "Server=.;Database=ITI;Integrated Security=true;TrustServerCertificate=True;";
```

- `Integrated Security=true` and `Integrated Security=SSPI` both mean Windows authentication.
- Microsoft examples also use `Trusted_Connection=True` for Windows authentication.
- If you also give a user name and password, they are **ignored** and Windows authentication is used.

### SQL Server authentication

```csharp
string cs = "Server=.;Database=ITI;User ID=myUser;Password=myPassword;TrustServerCertificate=True;";
```

### Named instance

Use `ServerName\InstanceName`. In C#, the backslash must be escaped:

```csharp
string cs1 = "Server=.\\SQLEXPRESS;Database=ITI;Integrated Security=true;";   // normal string
string cs2 = @"Server=.\SQLEXPRESS;Database=ITI;Integrated Security=true;";   // verbatim string
```

---

## Encrypt and TrustServerCertificate

- Since `Microsoft.Data.SqlClient` **4.0**, `Encrypt` is `true` by default.
- Encryption needs a **certificate on the server**. A local dev server often has no verifiable one.
- `TrustServerCertificate=True` encrypts the connection but **does not validate** the server certificate. This can expose you to man-in-the-middle attacks.

| Environment | Setting |
|---|---|
| Local development | `TrustServerCertificate=True` is acceptable |
| Production | Server has a verifiable certificate, `Encrypt=True`, `TrustServerCertificate=False` |

> Behavior of these keywords changed between driver versions. Check the docs for your version.

---

## Create, open, and close

```csharp
using Microsoft.Data.SqlClient;

using (var con = new SqlConnection(cs))
{
    con.Open();
    Console.WriteLine(con.State);   // Open
}   // closed automatically, even if an exception happens
```

- `using` is safer than calling `Close()` by hand.
- `Open()` throws `SqlException` if it cannot connect (wrong server, login failed, certificate problem).
- Open late, close early. Details: `resource-management-and-using.md` and `connection-pooling.md`.

---

## SqlConnectionStringBuilder

Builds a valid connection string without manual string concatenation. Its properties map to the connection string keywords.

```csharp
var builder = new SqlConnectionStringBuilder
{
    DataSource = ".",
    InitialCatalog = "ITI",
    IntegratedSecurity = true,
    TrustServerCertificate = true
};

string cs = builder.ConnectionString;
```

---

## Do not hard-code the connection string

### .NET Framework: `App.config` / `Web.config`

```xml
<configuration>
  <connectionStrings>
    <add name="DefaultConnection"
         connectionString="Server=.;Database=ITI;Integrated Security=true;TrustServerCertificate=True;" />
  </connectionStrings>
</configuration>
```

```csharp
using System.Configuration;   // add a reference to System.Configuration

string cs = ConfigurationManager.ConnectionStrings["DefaultConnection"].ConnectionString;
```

### Modern .NET (ASP.NET Core): `appsettings.json`

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=.;Database=ITI;Integrated Security=true;TrustServerCertificate=True;"
  }
}
```

```csharp
string cs = builder.Configuration.GetConnectionString("DefaultConnection");
```

> **Security:** Never commit real passwords. Prefer Windows authentication, user secrets, or environment variables. Keep `Persist Security Info=false` (the default) so the password cannot be read from the connection after it opens.

---

## Modern .NET differences

| Topic | .NET Framework | Modern .NET |
|---|---|---|
| Package | `System.Data.SqlClient` (built in) | `Microsoft.Data.SqlClient` (NuGet) |
| Configuration | `App.config` + `ConfigurationManager` | `appsettings.json` + `IConfiguration` |
| Connection string syntax | Same keywords | Same keywords, plus newer options in newer versions (for example `Encrypt=Strict` from 5.0) |

---

## Common mistakes

1. `Server=.\SQLEXPRESS` in a normal C# string (needs `\\` or `@""`).
2. Using `TrustServerCertificate=True` in production.
3. Hard-coding the connection string, or committing passwords to Git.
4. Not disposing the connection (no `using`).
5. Mixing `System.Data.SqlClient` and `Microsoft.Data.SqlClient` in one project.

---

## Summary

- Connection string = server + database + authentication (+ options).
- Prefer Windows authentication. Use `SqlConnectionStringBuilder` to build strings safely.
- Always open inside `using`. Keep strings in configuration, not in code.

---

## Next

[Resource Management and Using](resource-management-and-using.md)