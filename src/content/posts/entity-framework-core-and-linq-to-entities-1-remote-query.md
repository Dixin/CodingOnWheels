---
title: "Entity Framework Core and LINQ to Entities in Depth (1) Remote Query"
published: 2019-10-01
description: "The previous chapters discussed LINQ to Objects, LINQ to XML, and Parallel LINQ. All of these LINQ technologies query local in-memory objects managed by .NET. This chapter discusses a different kind o"
image: ""
tags: [".NET", ".NET Core", "C#", "EF Core", "Entity Framework Core", "LINQ", "LINQ to Entities", "SQL", "SQL Server"]
category: "Entity Framework Core"
draft: false
lang: "en"
---

> [!TIP]  
> [Functional Programming and LINQ via C#](/posts/linq-via-csharp) Series
>
> [Entity Framework Core](/archive/?tag=Entity%20Framework%20Core) Series

## Entity Framework Core

The previous chapters discussed LINQ to Objects, LINQ to XML, and Parallel LINQ. All of these LINQ technologies query local in-memory objects managed by .NET. This chapter discusses a different kind of LINQ technology, LINQ to Entities, which queries relational data managed by databases. LINQ to Entities was initially provided by Entity Framework (EF), a Microsoft library released since .NET Framework 3.5 Service Pack 1. Since 2016, Microsoft also released Entity Framework Core (EF Core), along with .NET Core. EF Core is based on .NET Standard, so it works cross-platform.

EF Core implements a provider model, so that LINQ to Entities can be implemented by different providers to work with different kinds of databases, including SQL Server (on-premise database) and Azure SQL Database (cloud database, aka SQL Azure), DB2, MySQL, Oracle, PostgreSQL, SQLLite, etc.

### SQL database

To demonstrate LINQ to Entities queries and other database operations, this book uses the classic sample SQL database AdventureWorks provided by Microsoft as the data source, because this sample database has a very intuitive structure, it also works with Azure SQL Database and all SQL Server editions. The full sample database provided by Microsoft is relatively large, so a trimmed version is provided in the code samples repo of this book:

-   The AdventureWorks.bacpac file is for Azure SQL Database
-   The `AdventureWorks_Data.mdf` and `AdventureWorks_Log.ldf` files are for SQL Server

There are many free options to setup SQL database. To setup in the cloud, follow these steps:

1.  Sign up [Azure free trial](https://azure.com/free) program, or sign up [Visual Studio Dev Essentials](https://www.visualstudio.com/dev-essentials/) program, to get free Azure account and free credits.
1.  Sign in to Azure portal, create a storage account, then create a container, and upload the above bacpac file into the container.
1.  In Azure portal, create a SQL Database server, then add local IP address to the server’s firewall settings to enable access.
1.  In Azure portal, import the uploaded bacpac file from the storage account to the server, and create a SQL database. There the many pricing tier options for the database creation, where the Basic tier starts from about $5 per month, which can be covered by the free credit.

As a alternative to cloud, SQL Server on premise can also be installed locally, then the above mdf and ldf files can be attached:

-   On Windows, there are several free options to install SQL Server:
    -   SQL Server LocalDB: the easiest option, with no configuration required for setup.
    -   SQL Server Express Core
    -   SQL Server Express with Advanced Services
    -   SQL Server Developer Edition: free after signing up [Visual Studio Dev Essentials](https://www.visualstudio.com/dev-essentials/) program
    -   SQL Server Evaluation for the next version
-   On Linux, SQL Server Express, Developer, and Evaluation editions are freely licensed.
-   On Mac, SQL Server can be installed using a Windows/Linux virtual machine, or Docker

After setting up, tools can be optionally installed to connect to and manage the SQL database:

-   On Windows, there are rich tools:
    -   [SQL Server Data Tools](https://msdn.microsoft.com/en-us/library/mt204009.aspx) for Visual Studio, a free Visual Studio extension enabling SQL database management inside Visual Studio
    -   [SQL Server Management Tools](https://msdn.microsoft.com/en-us/library/mt238290.aspx), which includes [SQL Server Management Studio](https://en.wikipedia.org/wiki/SQL_Server_Management_Studio) (a free integration environment to manage SQL database), [SQL Server Profiler](https://msdn.microsoft.com/en-us/library/ms181091.aspx) (a free tracing tool for SQL Server on premise), and other tools.
-   On Windows, Linux, and macOS:
    -   SQL Server (mssql) for Visual Studio Code, an extension for Visual Studio Code to execute SQL
    -   Azure Data Studio, a free cross-platform tool to manage data and edit query.

To connect to the sample database, its connection string can be saved in the configuration of application or service during development and test. For .NET Core, the connection string can be saved for the application as a JSON file, for example, as app.json file:

```json
{
"ConnectionStrings": {
"AdventureWorks": "Server=tcp:dixin.database.windows.net,1433;Initial Catalog=AdventureWorks;Persist Security Info=False;User ID=***;Password=***;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"
}
}
```

For .NET Framework, the connection string can be saved in the application’s app.config file:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
<connectionStrings>
<add name="AdventureWorks" connectionString="Server=tcp:dixin.database.windows.net,1433;Initial Catalog=AdventureWorks;Persist Security Info=False;User ID=***;Password=***;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;" />
</connectionStrings>
</configuration>
```

Then the connection string can be loaded and used in C# code:

```csharp
internal static class ConnectionStrings
{
internal static string AdventureWorks { get; } =
#if NETFX
ConfigurationManager.ConnectionStrings[nameof(AdventureWorks)].ConnectionString;
#else
new ConfigurationBuilder().AddJsonFile("App.json").Build()
.GetConnectionString(nameof(AdventureWorks));
#endif
}
```

The connection string for production should be protected with encryption or tools like Azure Key Vault configuration provider.

### Remote query vs. local query

LINQ to Objects, Parallel LINQ query .NET objects in current .NET application’s local memory, these queries are called local queries. LINQ to XML queries XML data source, which are local .NET objects representing XML structures as well, so LINQ to XML queries are also local queries. As demonstrated at the beginning of this book, LINQ can also query data in other data domains, like tweets in Twitter, rows in database tables, etc. Apparently, these data source are not .NET objects directly available in local memory. These queries are called remote queries.

Remote LINQ (like LINQ to Entities) is provided as paraty of local LINQ (like LINQ to Objects). Since local data sources and local queries are represented by `IEnumerable<T>`, remote LINQ data sources (like a table in database) and remote queries (like a database query), are represented by `System.Linq.IQueryable<T>`:

| LINQ to (local) Objects                     | LINQ to (remote) Entities          |
|---------------------------------------------|------------------------------------|
| `System.Collections.IEnumerable`            | `System.Linq.IQueryable`           |
| `System.Collections.Generic.IEnumerable<T>` | `System.Linq.IQueryable<T>`        |
| `System.Linq.IOrderedEnumerable<T>`         | `System.Linq.IOrderedQueryable<T>` |
| `System.Linq.Enumerable`                    | `System.Linq.Queryable`            |

```csharp
namespace System.Linq
{
public interface IQueryable : IEnumerable
{
Expression Expression { get; }

Type ElementType { get; }

IQueryProvider Provider { get; }
}

public interface IOrderedQueryable : IQueryable, IEnumerable { }

public interface IQueryable<out T> : IEnumerable<T>, IEnumerable, IQueryable { }

public interface IOrderedQueryable<out T> : IQueryable<T>, IEnumerable<T>, IOrderedQueryable, IQueryable, IEnumerable { }
}
```

.NET Standard and Microsoft libraries provide many implementation of `IEnumerable<T>`, like `T[]` representing array, `List<T>` representing mutable list, `Microsoft.Collections.Immutable.ImmutableList<T>` representing immutable list, etc. EF Core also provides implementation of `IQueryable<T>`, including `Microsoft.EntityFrameworkCore.DbSet<T>` representing database table, `Microsoft.EntityFrameworkCore.Query.Internal.EntityQueryable<T>` representing database query, etc.

As the parity with System.Linq.Enumerable, System.Linq.Queryable static type provides the remote version of standard queries. For example, the following are the local and remote Where/Select/Concat/Cast queries side by side:

```csharp
namespace System.Linq
{
public static class Enumerable
{
public static IEnumerable<TSource> Where<TSource>(
this IEnumerable<TSource> source, Func<TSource, bool> predicate);

public static IEnumerable<TResult> Select<TSource, TResult>(
this IEnumerable<TSource> source, Func<TSource, TResult> selector);

public static IEnumerable<TSource> Concat<TSource>(
this IEnumerable<TSource> first, IEnumerable<TSource> second);

public static IEnumerable<TResult> Cast<TResult>(this IEnumerable source);

// Other members.
}

public static class Queryable
{
public static IQueryable<TSource> Where<TSource>(
this IQueryable<TSource> source, Expression<Func<TSource, bool>> predicate);

public static IQueryable<TResult> Select<TSource, TResult>(
this IQueryable<TSource> source, Expression<Func<TSource, TResult>> selector);

public static IQueryable<TSource> Concat<TSource>(
this IQueryable<TSource> source1, IEnumerable<TSource> source2);

public static IQueryable<TResult> Cast<TResult>(this IQueryable source);

// Other members.
}
}
```

When defining each standard query in remote LINQ, the generic source and generic output are represented by `IQueryable<T>` instead of `IEnumerable<T>`, and the non-generic source is represented by IQueryable instead of IEnumerable. The iteratee functions are replaced by expression trees. Similarly, the following are the ordering queries side by side, where the ordered source and ordered output are represented by `IOrderedQueryable<T>` instead of `IOrderedEnumerable<T>`:

```csharp
namespace System.Linq
{
public static class Enumerable
{
public static IOrderedEnumerable<TSource> OrderBy<TSource, TKey>(
this IEnumerable<TSource> source, Func<TSource, TKey> keySelector);

public static IOrderedEnumerable<TSource> OrderByDescending<TSource, TKey>(
this IEnumerable<TSource> source, Func<TSource, TKey> keySelector);

public static IOrderedEnumerable<TSource> ThenBy<TSource, TKey>(
this IOrderedEnumerable<TSource>source, Func<TSource, TKey> keySelector);

public static IOrderedEnumerable<TSource> ThenByDescending<TSource, TKey>(
this IOrderedEnumerable<TSource> source, Func<TSource, TKey> keySelector);
}

public static class Queryable
{
public static IOrderedQueryable<TSource> OrderBy<TSource, TKey>(
this IQueryable<TSource> source, Expression<Func<TSource, TKey>> keySelector);

public static IOrderedQueryable<TSource> OrderByDescending<TSource, TKey>(
this IQueryable<TSource> source, Expression<Func<TSource, TKey>> keySelector);

public static IOrderedQueryable<TSource> ThenBy<TSource, TKey>(
this IOrderedQueryable<TSource> source, Expression<Func<TSource, TKey>> keySelector);

public static IOrderedQueryable<TSource> ThenByDescending<TSource, TKey>(
this IOrderedQueryable<TSource> source, Expression<Func<TSource, TKey>> keySelector);
}
}
```

With this design, the fluent function chaining and the LINQ query expression pattern are automatically enabled for remote LINQ queries. It is the same syntax to write LINQ to Objects query and remote LINQ query.

Queryable does not provide the following queries:

-   Empty/Range/Repeat: it does not make sense for .NET to locally generate a remote data source or remote query on the fly; the other generation query DefaultIfEmpty is available, because DefaultIfEmpty works with an existing `IQueryable<T>` source.
-   AsEnumerable: Enumerable.AsEnumerable types any `IEnumerable<T>` source just as `IEnumerable<T>`. Since `IQueryable<T>` implements `IEnumerable<T>`, Enumerable.AsEnumerable also works for `IQueryanle<T>`.
-   ToArray/ToDictionary/ToList/ToLookup: LINQ to Objects provides these colection queries to pull values from any `IEnumerable<T>`source and create local .NET collections. Since `IQueryable<T>` implements `IEnumerable<T>`, these queries provided by LINQ to Objects also works for `IQueryanle<T>`.
-   Max/Min overloads for .NET primary types: these are specific types of local .NET application, not the remote data domain.

Queryable also provides an additional query AsQueryable, as the paraty with AsEnumerable. However, unlike AsSequential/AsParallel switching between sequential and parallel query, AsEnumerable/AsQueryable cannot freely switch between local and remote query. This query is discussed later.

### Function vs. expression tree

Enumerable queries accept iteratee functions, and Queryable queries accept expression trees. As discussed in the lamda expression chapter, functions are executable .NET code, and expression trees are data structures representing the abstract syntax tree of functions, which can be translated to other domain-specific language. The lambda expression chapter also demonstrates compiling an arithmetic expression tree to CIL code at runtime, and executing it dynamically. The same approach can be used to translate arithmetic expression tree to SQL query, and execute it in a remote SQL database. The following function traverses an arithmetic expression tree with +, -, \*, / operators, and compile it to a SQL SELECT statement with infix arithmetic expression:

```csharp
internal static string InOrder(this LambdaExpression expression)
{
string VisitNode(Expression node)
{
switch (node.NodeType)
{
case ExpressionType.Constant when node is ConstantExpression constant:
return constant.Value.ToString();

case ExpressionType.Parameter when node is ParameterExpression parameter:
return $"@{parameter.Name}";

// In-order output: left child, current node, right child.
case ExpressionType.Add when node is BinaryExpression binary:
return $"({VisitNode(binary.Left)} + {VisitNode(binary.Right)})";

case ExpressionType.Subtract when node is BinaryExpression binary:
return $"({VisitNode(binary.Left)} - {VisitNode(binary.Right)})";

case ExpressionType.Multiply when node is BinaryExpression binary:
return $"({VisitNode(binary.Left)} * {VisitNode(binary.Right)})";

case ExpressionType.Divide when node is BinaryExpression binary:
return $"({VisitNode(binary.Left)} / {VisitNode(binary.Right)})";

default:
throw new ArgumentOutOfRangeException(nameof(expression));
}
}
return $"SELECT {VisitNode(expression.Body)};";
}
```

Here @ is prepended to each parameter name, which is the SQL syntax. The following code demonstrates the compilation:

```csharp
internal static void Infix()
{

Expression<Func<double, double, double, double, double, double>> expression =
(a, b, c, d, e) => a + b - c * d / 2D + e * 3D;
string sql = expression.InOrder();
sql.WriteLine(); // SELECT (((@a + @b) - ((@c * @d) / 2)) + (@e * 3));
}
```

The following ExecuteSql function is defined to execute the compiled SQL statement with SQL parameters and SQL database connection string provided, and return the execution result from SQL database:

```csharp
internal static double ExecuteSql(
string connection,
string sql,
IDictionary<string, double> parameters)
{
using (SqlConnection sqlConnection = new SqlConnection(connection))
using (SqlCommand sqlCommand = new SqlCommand(sql, sqlConnection))
{
sqlConnection.Open();
parameters.ForEach(parameter => sqlCommand.Parameters.AddWithValue(parameter.Key, parameter.Value));
return (double)sqlCommand.ExecuteScalar();
}
}
```

And the following TranslateToSql function is defined to wrap the entire work. It accept an arithmetic expression tree, call the above InOrder to compile it to SQL, then emit a dynamic function, which extracts the parameters and calls above ExecuteScalar function to execute the SQL:

```csharp
public static TDelegate TranslateToSql<TDelegate>(
this Expression<TDelegate> expression, string connection)
{
DynamicMethod dynamicMethod = new DynamicMethod(
string.Empty,
expression.ReturnType,
expression.Parameters.Select(parameter => parameter.Type).ToArray(),
MethodBase.GetCurrentMethod().Module);
EmitCil(dynamicMethod.GetILGenerator(), expression.InOrder());
return (TDelegate)(object)dynamicMethod.CreateDelegate(typeof(TDelegate));

void EmitCil(ILGenerator generator, string sql)
{
// Dictionary<string, double> dictionary = new Dictionary<string, double>();
generator.DeclareLocal(typeof(Dictionary<string, double>));
generator.Emit(
OpCodes.Newobj,
typeof(Dictionary<string, double>).GetConstructor(Array.Empty<Type>()));
generator.Emit(OpCodes.Stloc_0);

for (int index = 0; index < expression.Parameters.Count; index++)
{
// dictionary.Add($"@{expression.Parameters[i].Name}", args[i]);
generator.Emit(OpCodes.Ldloc_0); // dictionary.
generator.Emit(OpCodes.Ldstr, $"@{expression.Parameters[index].Name}");
generator.Emit(OpCodes.Ldarg_S, index);
generator.Emit(
OpCodes.Callvirt,
typeof(Dictionary<string, double>).GetMethod(
nameof(Dictionary<string, double>.Add),
BindingFlags.Instance | BindingFlags.Public | BindingFlags.InvokeMethod));
}

// ExecuteSql(connection, expression, dictionary);
generator.Emit(OpCodes.Ldstr, connection);
generator.Emit(OpCodes.Ldstr, sql);
generator.Emit(OpCodes.Ldloc_0);
generator.Emit(
OpCodes.Call,
new Func<string, string, IDictionary<string, double>, double>(ExecuteSql).Method);

generator.Emit(OpCodes.Ret); // Returns the result.
}
}
```

As fore mentioned, .NET built-in `Expression<TDelegate>.Compile` method compiles expression tree to CIL, and emits a function to execute the CIL locally with current .NET application process. In contrast, here TranslateToSql compiles the arithmetic expression tree to SQL query, and emits a function to execute the SQL in a specified remote SQL database:

```csharp
internal static void TranslateAndExecute()
{
Expression<Func<double, double, double, double, double, double>> expression =
(a, b, c, d, e) => a + b - c * d / 2D + e * 3D;
Func<double, double, double, double, double, double> local = expression.Compile();
local(1, 2, 3, 4, 5).WriteLine(); // 12
Func<double, double, double, double, double, double> remote = expression.TranslateToSql(ConnectionStrings.AdventureWorks);
remote(1, 2, 3, 4, 5).WriteLine(); // 12
}
```
