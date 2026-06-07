---
title: "Understanding LINQ to SQL (9) Concurrent Conflict"
published: 2010-04-26
description: "Conflicts are very common when concurrently accessing the same data."
image: ""
tags: [".NET", "C#", "LINQ", "LINQ to SQL", "LINQ via C# Series", "SQL Server", "TSQL", "Visual Studio"]
category: "LINQ"
draft: false
lang: ""
---

> [!TIP]  
> [Functional Programming and LINQ via C#](/posts/linq-via-csharp) Series
>
> [LINQ to SQL](/archive/?tag=LINQ%20to%20SQL) Series
>
> [Entity Framework Core](/archive/?tag=Entity%20Framework%20Core) Series
>
> [Entity Framework](/archive/?tag=Entity%20Framework) Series

Conflicts are very common when [concurrently](http://en.wikipedia.org/wiki/Concurrency_\(computer_science\)) accessing the same data.

## Conflicts in concurrent data access

The following code demonstrates the concurrent conflict scenario:

```csharp
Action<int, Action<Category>> updateCategory = (id, updater) =>
    {
        using (NorthwindDataContext database = new NorthwindDataContext())
        {
            Category category = database.Categories
                                        .Single(item => item.CategoryID == id);

            Thread.Sleep(4000);

            updater(category);
            // database.SubmitChanges() invokes:
            database.SubmitChanges(ConflictMode.FailOnFirstConflict);
        }
    };

new Thread(() => updateCategory(1, category => category.CategoryName = "Thread 1")).Start();

Thread.Sleep(2000);

new Thread(() => updateCategory(1, category => category.CategoryName = "Thread 2")).Start();
```

Here 2 threads are accessing the same category. This is the order of the executions:

| Time (second)       | Thread 1                          | Thread 2                                | `[CategoryName]` database value           |
|---------------------|-----------------------------------|-----------------------------------------|-------------------------------------------|
| 0 (Thread 1 reads)  | Retrieves “Beverages”             |                                         | “Beverages”                               |
| 2 (Thread 2 reads)  |                                   | Retrieves “Beverages”                   | “Beverages”                               |
| 4 (Thread 1 writes) | Updates “Beverages” to “Thread 1” |                                         | “Thread 1”                                |
| 6 (Thread 2 writes) |                                   | Should update “Beverages” to “Thread 2” | `[CategoryName]` is no longer “Beverages” |

When the later started thread (thread 2) tries to submit the change, the conflict occurs, and DataContext.SubmitChanges() throws a ChangeConflictException:

> Row not found or changed.

## Optimistic concurrency control

The [concurrency control](http://en.wikipedia.org/wiki/Concurrency_control) tactic of LINQ to SQL is [optimistic](http://en.wikipedia.org/wiki/Optimistic_concurrency_control), which means LINQ to SQL checks the status of data instead of locking the data (pessimistic concurrency control).

This is the translated SQL from 2 threads:

```sql
-- Thread 1 reads.
exec sp_executesql N'SELECT [t0].[CategoryID], [t0].[CategoryName], [t0].[Description], [t0].[Picture]
FROM [dbo].[Categories] AS [t0]
WHERE [t0].[CategoryID] = @p0',N'@p0 int',@p0=1

-- Thread 2 reads.
exec sp_executesql N'SELECT [t0].[CategoryID], [t0].[CategoryName], [t0].[Description], [t0].[Picture]
FROM [dbo].[Categories] AS [t0]
WHERE [t0].[CategoryID] = @p0',N'@p0 int',@p0=1

-- Thread 1 writes.
BEGIN TRANSACTION 
exec sp_executesql N'UPDATE [dbo].[Categories]
SET [CategoryName] = @p2
WHERE ([CategoryID] = @p0) AND ([CategoryName] = @p1)',N'@p0 int,@p1 nvarchar(4000),@p2 nvarchar(4000)',@p0=1,@p1=N'Beverages',@p2=N'Thread 1' -- CategoryName has an [Column(UpdateCheck = UpdateCheck.Always)] attribute.
COMMIT TRANSACTION -- Updating successes.

-- Thread 2 writes.
BEGIN TRANSACTION 
exec sp_executesql N'UPDATE [dbo].[Categories]
SET [CategoryName] = @p2
WHERE ([CategoryID] = @p0) AND ([CategoryName] = @p1)',N'@p0 int,@p1 nvarchar(4000),@p2 nvarchar(4000)',@p0=1,@p1=N'Beverages',@p2=N'Thread 2' -- CategoryName has an [Column(UpdateCheck = UpdateCheck.Always)] attribute.
ROLLBACK TRANSACTION -- Updating fails.
```

When submitting data changes, LINQ to SQL not only uses primary key to identify the data, but also checks the original state of the column which is expected to be updated.

### Update check

This original state check is specified by the `[Column]` attribute of entity property:

![image](https://aspblogs.z22.web.core.windows.net/dixin/Media/image_14545167.png "image")

If ColumnAttribute.UpdateCheck is not specified:

```csharp
[Column(Storage = "_CategoryName", DbType = "NVarChar(15) NOT NULL", CanBeNull = false)]
public string CategoryName
{
}
```

then it will have a default value: UpdateCheck.Always:

```csharp
[AttributeUsage(AttributeTargets.Field | AttributeTargets.Property, AllowMultiple = false)]
public sealed class ColumnAttribute : DataAttribute
{
    private UpdateCheck _updateCheck = UpdateCheck.Always;

    public UpdateCheck UpdateCheck
    {
        get
        {
            return this._updateCheck;
        }
        set
        {
            this._updateCheck = value;
        }
    }
}
```

### Time stamp

In the above screenshot, there is a `[Time Stamp]` option in the O/R designer, which can be used when this column is of type timestamp (rowversion). To demonstrating this, add a timestamp column `[Version]` to the `[Categories]` table:

![image](https://aspblogs.z22.web.core.windows.net/dixin/Media/image_2F5CFAA8.png "image")

And recreate the model in O/R designer. Now this is the generated `[Column]` attribute:

```csharp
[Column(Storage = "_Version", AutoSync = AutoSync.Always, DbType = "rowversion NOT NULL", 
    CanBeNull = false, IsDbGenerated = true, IsVersion = true, UpdateCheck = UpdateCheck.Never)]
public Binary Version
{
}
```

Now LINQ to SQL always checks the `[Version]` column instead of `[CategoryName]` column. So when rerunning the above code, the translated SQL is different:

```sql
-- Thread 1 reads.
exec sp_executesql N'SELECT [t0].[CategoryID], [t0].[CategoryName], [t0].[Description], [t0].[Picture], [t0].[Version]
FROM [dbo].[Categories] AS [t0]
WHERE [t0].[CategoryID] = @p0',N'@p0 int',@p0=1

-- Thread 2 reads.
exec sp_executesql N'SELECT [t0].[CategoryID], [t0].[CategoryName], [t0].[Description], [t0].[Picture], [t0].[Version]
FROM [dbo].[Categories] AS [t0]
WHERE [t0].[CategoryID] = @p0',N'@p0 int',@p0=1

-- Thread 1 writes.
BEGIN TRANSACTION 
-- Checks time stamp.
exec sp_executesql N'UPDATE [dbo].[Categories]
SET [CategoryName] = @p2
WHERE ([CategoryID] = @p0) AND ([Version] = @p1)

SELECT [t1].[Version]
FROM [dbo].[Categories] AS [t1]
WHERE ((@@ROWCOUNT) > 0) AND ([t1].[CategoryID] = @p3)',N'@p0 int,@p1 timestamp,@p2 nvarchar(4000),@p3 int',@p0=1,@p1=0x0000000000000479,@p2=N'Thread 1',@p3=1
-- SELECT for [Column(AutoSync = AutoSync.Always)]
COMMIT TRANSACTION -- Updating successes.

-- Thread 2 writes.
BEGIN TRANSACTION 
-- Checks time stamp.
exec sp_executesql N'UPDATE [dbo].[Categories]
SET [CategoryName] = @p2
WHERE ([CategoryID] = @p0) AND ([Version] = @p1)

SELECT [t1].[Version]
FROM [dbo].[Categories] AS [t1]
WHERE ((@@ROWCOUNT) > 0) AND ([t1].[CategoryID] = @p3)',N'@p0 int,@p1 timestamp,@p2 nvarchar(4000),@p3 int',@p0=1,@p1=0x0000000000000479,@p2=N'Thread 2',@p3=1
-- SELECT for [Column(AutoSync = AutoSync.Always)]
ROLLBACK TRANSACTION -- Updating fails.
```

## Handle ChangeConflictException

When concurrent conflict occurs, SubmitChanges() rollbacks the TRANSACTION, then throws a ChangeConflictException exception.

So if the caller of DataContext.SubmitChanges() knows how to resolve the conflict, it can detects conflict by handling ChangeConflictException .

### Merge changes to resolve conflict

For example, a common tactic is to merge the changes into database:

```csharp
Action<int, Action<Category>> updateCategory = (id, updater) =>
    {
        using (NorthwindDataContext database = new NorthwindDataContext())
        {
            Category category = database.Categories
                                        .Single(item => item.CategoryID == id);

            Thread.Sleep(4000);

            updater(category);
            try
            {
                // All data changes will be tried before rollback.
                database.SubmitChanges(ConflictMode.ContinueOnConflict);
                // Now all conflicts are stored in DataContext.ChangeConflicts.
            }
            catch (ChangeConflictException)
            {
                foreach (ObjectChangeConflict conflict in database.ChangeConflicts)
                {
                    Console.WriteLine(
                        "Conflicted row: ID = {0}.",
                        (conflict.Object as Category).CategoryID);

                    foreach (MemberChangeConflict member in conflict.MemberConflicts)
                    {
                        Console.WriteLine(
                            "[{0}] column is expected to be '{1}' in database, but it is not.",
                            member.Member.Name,
                            member.CurrentValue);
                    }

                    conflict.Resolve(RefreshMode.KeepChanges); // Queries row to merge changes.
                    Console.WriteLine("Merged changes to row: {0}.", conflict.IsResolved);
                }

                // Submits again by merging changes.
                database.SubmitChanges();
            }
        }
    };

new Thread(() => updateCategory(1, category => category.CategoryName = "Thread 1")).Start();

Thread.Sleep(2000);

new Thread(() => updateCategory(1, category => category.Description = "Thread 2")).Start();
```

Running this refined code will print:

> Conflicted row: ID = 1. `[CategoryName]` column is expected to be 'Beverages' in database, but it is not. Merged changes to row: True.

This is the order of the executions:

|Time (second)|Thread 1|Thread 2|`[CategoryName]`|`[Description]`|
|---|---|---|---|---|
|0|Retrieves “Beverages” for `[CategoryName]`.||“Beverages”|“Soft drinks, coffees, teas, beers, and ales”|
|2||Retrieves “Beverages” for `[CategoryName]`.|“Beverages”|“Soft drinks, coffees, teas, beers, and ales”|
|4|Checks whether `[CategoryName]` is “Beverages”, and updates `[CategoryName]`.||“Thread 1”|“Soft drinks, coffees, teas, beers, and ales”|
|6||Checks whether `[CategoryName]` is “Beverages”.|“Thread 1”|“Soft drinks, coffees, teas, beers, and ales”|
|||Retrieves “Thread1” for `[CategoryName]`|“Thread 1”|“Soft drinks, coffees, teas, beers, and ales”|
|||Checks whether `[CategoryName]` is “Thread 1”., and updates `[Description]`.|“Thread 1”|“Thread 2”|

Please notice that, to merge the changes, database must be queried.

This is the entire translated SQL:

```sql
-- Thread 1 reads.
exec sp_executesql N'SELECT [t0].[CategoryID], [t0].[CategoryName], [t0].[Description], [t0].[Picture]
FROM [dbo].[Categories] AS [t0]
WHERE [t0].[CategoryID] = @p0',N'@p0 int',@p0=1

-- Thread 2 reads.
exec sp_executesql N'SELECT [t0].[CategoryID], [t0].[CategoryName], [t0].[Description], [t0].[Picture]
FROM [dbo].[Categories] AS [t0]
WHERE [t0].[CategoryID] = @p0',N'@p0 int',@p0=1

-- Thread 1 writes.
BEGIN TRANSACTION 
exec sp_executesql N'UPDATE [dbo].[Categories]
SET [CategoryName] = @p2
WHERE ([CategoryID] = @p0) AND ([CategoryName] = @p1)',N'@p0 int,@p1 nvarchar(4000),@p2 nvarchar(4000)',@p0=1,@p1=N'Beverages',@p2=N'Thread 1' -- CategoryName has an [Column(UpdateCheck = UpdateCheck.Always)] attribute.
COMMIT TRANSACTION -- Updating successes.

-- Thread 2 writes.
BEGIN TRANSACTION 
exec sp_executesql N'UPDATE [dbo].[Categories]
SET [Description] = @p2
WHERE ([CategoryID] = @p0) AND ([CategoryName] = @p1)',N'@p0 int,@p1 nvarchar(4000),@p2 ntext',@p0=1,@p1=N'Beverages',@p2=N'Thread 2' -- CategoryName has an [Column(UpdateCheck = UpdateCheck.Always)] attribute.
ROLLBACK TRANSACTION -- Updating fails.

-- Thread 2 reads data to merge changes.
exec sp_executesql N'SELECT [t0].[CategoryID], [t0].[CategoryName], [t0].[Description], [t0].[Picture]
FROM [dbo].[Categories] AS [t0]
WHERE [t0].[CategoryID] = @p0',N'@p0 int',@p0=1

-- Thread 2 writes again.
BEGIN TRANSACTION 
exec sp_executesql N'UPDATE [dbo].[Categories]
SET [CategoryName] = @p2, [Description] = @p3
WHERE ([CategoryID] = @p0) AND ([CategoryName] = @p1)',N'@p0 int,@p1 nvarchar(4000),@p2 nvarchar(4000),@p3 ntext',@p0=1,@p1=N'Thread 1',@p2=N'Thread 1',@p3=N'Thread 2'
COMMIT TRANSACTION -- Updating successes.
```

To resolve conflicts, an easier way is just invoking ChangeConflictCollection.ResolveAll():

```csharp
catch (ChangeConflictException)
{
    database.ChangeConflicts.ResolveAll(RefreshMode.KeepChanges);
    database.SubmitChanges();
}
```

## More about concurrency

Because this is a LINQ / functional programming series, not a SQL / database series, this post only gives a brief explanation about how LINQ to SQL controls concurrent conflict. please check [MSDN](http://msdn.microsoft.com/en-us/library/bb399373.aspx) and Wikipedia for further topics, like [concurrency](http://en.wikipedia.org/wiki/Concurrency_\(computer_science\)), [concurrency control](http://en.wikipedia.org/wiki/Concurrency_control), [optimistic concurrency control](http://en.wikipedia.org/wiki/Optimistic_concurrency_control), [timestamp-based concurrency control](http://en.wikipedia.org/wiki/Timestamp-based_concurrency_control), [SQL Server transactions](http://msdn.microsoft.com/en-us/library/aa213068\(v=SQL.80\).aspx), [SQL Server locking](http://msdn.microsoft.com/en-us/library/aa213039\(SQL.80\).aspx), [SQL Server isolation levels](http://msdn.microsoft.com/en-us/library/aa213034\(SQL.80\).aspx), [SQL Server row level versioning](http://msdn.microsoft.com/en-us/library/cc917674.aspx), etc.
