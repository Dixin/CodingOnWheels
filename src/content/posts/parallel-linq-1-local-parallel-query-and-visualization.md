---
title: "Parallel LINQ in Depth (1) Local Parallel Query and Visualization"
published: 2019-09-20
description: "LINQ to Objects and LINQ to XML queries are designed to work sequentially, and do not involve multi-threading, concurrency, or parallel computing. To scale LINQ query in multi-processor environment, ."
image: ""
tags: [".NET", "C#", "Concurrency VIsualizer", "LINQ", "Parallel Computing", "Parallel LINQ", "PLINQ"]
category: "Parallel LINQ"
draft: false
lang: "en"
---

> [!TIP]
> [Functional Programming and LINQ via C#](/posts/linq-via-csharp) Series
>
> [Parallel LINQ in Depth](/archive/?tag=Parallel%20LINQ) Series

LINQ to Objects and LINQ to XML queries are designed to work sequentially, and do not involve multi-threading, concurrency, or parallel computing. To scale LINQ query in multi-processor environment, .NET Standard provides parallel version of LINQ to Objects, called Parallel LINQ or PLINQ.

## Parallel LINQ query

Parallel LINQ (to Objects) APIs are provided as a parity with (sequential) LINQ to Objects APIs:

| LINQ to Objects types                       | PLINQ types                           |
|---------------------------------------------|---------------------------------------|
| `System.Collections.IEnumerable`            | `System.Linq.ParallelQuery`           |
| `System.Collections.Generic.IEnumerable<T>` | `System.Linq.ParallelQuery<T>`        |
| `System.Linq.IOrderedEnumerable<T>`         | `System.Linq.OrderedParallelQuery<T>` |
| `System.Linq.Enumerable`                    | `System.Linq.ParallelEnumerable`      |

As the parity with System.Linq.Enumerable, System.Linq.ParallelEnumerable static type provides the parallel version of standard queries. For example, the following is the comparison of the Range/Repeat generation queries’ sequential and parallel versions:

```csharp
namespace System.Linq
{
public static class Enumerable
{
public static IEnumerable<int> Range(int start, int count);

public static IEnumerable<TResult> Repeat<TResult>(TResult element, int count);

// Other members.
}

public static class ParallelEnumerable
{
public static ParallelQuery<int> Range(int start, int count);

public static ParallelQuery<TResult> Repeat<TResult>(TResult element, int count);

// Other members.
}
}
```

And the following are the sequential and parallel Where/Select/Concat/Cast queries side by side:

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
}

public static class ParallelEnumerable
{
public static ParallelQuery<TSource> Where<TSource>(
this ParallelQuery<TSource> source, Func<TSource, bool> predicate);

public static ParallelQuery<TResult> Select<TSource, TResult>(
this ParallelQuery<TSource> source, Func<TSource, TResult> selector);

public static ParallelQuery<TSource> Concat<TSource>(
this ParallelQuery<TSource> first, ParallelQuery<TSource> second);

public static ParallelQuery<TResult> Cast<TResult>(this ParallelQuery source);
}
}
```

When defining each standard query in PLINQ, the generic source and generic output are represented by `ParallelQuery<T>` instead of `IEnumerable<T>`, and the non-generic source is represented by ParallelQuery instead of IEnumerable. The other parameter types remain the same. Similarly, the following are the ordering queries side by side, where the ordered source and ordered output are represented by `OrderedParallelQuery<T>` instead of `IOrderedEnumerable<T>`:

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
this IOrderedEnumerable<TSource> source, Func<TSource, TKey> keySelector);

public static IOrderedEnumerable<TSource> ThenByDescending<TSource, TKey>(
this IOrderedEnumerable<TSource> source, Func<TSource, TKey> keySelector);
}

public static class ParallelEnumerable
{
public static OrderedParallelQuery<TSource> OrderBy<TSource, TKey>(
this ParallelQuery<TSource> source, Func<TSource, TKey> keySelector);

public static OrderedParallelQuery<TSource> OrderByDescending<TSource, TKey>(
this ParallelQuery<TSource> source, Func<TSource, TKey> keySelector);

public static OrderedParallelQuery<TSource> ThenBy<TSource, TKey>(
this OrderedParallelQuery<TSource> source, Func<TSource, TKey> keySelector);

public static OrderedParallelQuery<TSource> ThenByDescending<TSource, TKey>(
this OrderedParallelQuery<TSource> source, Func<TSource, TKey> keySelector);
}
}
```

With this design, the fluent function chaining and the LINQ query expression pattern are automatically enabled for PLINQ queries. It is the same syntax to write LINQ to Objects query and PLINQ query.

Besides the parities with Enumerable queries, ParallelEnumerable also provides additional queries and additional overloads for Aggregate query:

-   Sequence queries
    -   Conversion: AsParallel, AsSequential
    -   Query settings: WithCancellation, WithDegreeOfParallelism, WithExecutionMode, WithMergeOptions
    -   Ordering: AsOrdered, AsUnordered
-   Value queries
    -   Aggregation: Aggregate
-   Void queries
    -   Iteration: ForAll

### Parallel query vs. sequential query

A `ParallelQuery<T>` source can be created by calling generation queries provided by ParallelEnumerable, like Range, Repeat, etc., then the other parallel queries can be used subsequently:

```csharp
internal static void Generation()
{
IEnumerable<double>sequentialQuery = Enumerable
.Repeat(0, 5) // Output IEnumerable<int>.
.Concat(Enumerable.Range(0, 5)) // Call Enumerable.Concat.
.Where(int32 => int32 > 0) // Call Enumerable.Where.
.Select(int32 => Math.Sqrt(int32)); // Call Enumerable.Select.

ParallelQuery<double> parallelQuery = ParallelEnumerable
.Repeat(0, 5) // Output ParallelQuery<int>.
.Concat(ParallelEnumerable.Range(0, 5)) // Call ParallelEnumerable.Concat.
.Where(int32 => int32 > 0) // Call ParallelEnumerable.Where.
.Select(int32 => Math.Sqrt(int32)); // Call ParallelEnumerable.Select.
}
```

A PLINQ query can also be started by calling ParallelEnumerable.AsParallel to convert `IEnumerable<T>`/`IEnumerable` to `ParallelQuery<T>`/`ParallelQuery`:

public static ParallelQuery AsParallel(this IEnumerable source);

```csharp
public static ParallelQuery<TSource> AsParallel<TSource>(this IEnumerable<TSource> source);
```

For example,

```csharp
internal static void AsParallel(IEnumerable<int> source1, IEnumerable source2)
{
ParallelQuery<int>parallelQuery1 = source1 // IEnumerable<int>.
.AsParallel(); // Output ParallelQuery<int>.

ParallelQuery<int> parallelQuery2 = source2 // IEnumerable.
.AsParallel() // Output ParallelQuery.
.Cast<int>(); // Call ParallelEnumerable.Cast.
}
```

AsParallel also has an overload accepting a partitioner. Partitioner is discussed in the next chapter.

To use sequential queries for a `ParallelQuery<T>` source, just call ParallelEnumerable.AsSequential or ParallelEnumerable.AsEnumerable to convert `ParallelQuery<T>` to `IEnumerable<T>`, then the sequential queries can be used subsequently:

```csharp
public static IEnumerable<TSource> AsSequential<TSource>(
this ParallelQuery<TSource> source);

public static IEnumerable<TSource> AsEnumerable<TSource>(
this ParallelQuery<TSource> source);
```

ParallelEnumerable.AsEnumerable simply calls AsSequential internally, so they are identical. For example:

```csharp
internal static partial class QueryMethods
{
private static readonly Assembly CoreLibrary = typeof(object).Assembly;

internal static void SequentialParallel()
{
IEnumerable<string> obsoleteTypes = CoreLibrary.GetExportedTypes() // Output IEnumerable<Type>.
.AsParallel() // Output ParallelQuery<Type>.
.Where(type => type.GetCustomAttribute<ObsoleteAttribute>() != null) // Call ParallelEnumerable.Where.
.Select(type => type.FullName) // Call ParallelEnumerable.Select.
.AsSequential() // Output IEnumerable<Type>.
.OrderBy(name => name); // Call Enumerable.OrderBy.
obsoleteTypes.WriteLines();
}
}
```

The above query can be written in query expression syntax:

```csharp
internal static void QueryExpression()
{
IEnumerable<string>obsoleteTypes =
from name in
(from type in CoreLibrary.GetExportedTypes().AsParallel()
where type.GetCustomAttribute<ObsoleteAttribute>() != null
select type.FullName).AsSequential()
orderby name
select name;
obsoleteTypes.WriteLines();
}
```

### Parallel query execution

The foreach statement or the EnumerableEx.ForEach query provided by Ix can be used to sequentially pull the results and start LINQ to Objects query execution. Their parallel version is the ParallelEnumerable.ForAll query.

```csharp
namespace System.Linq
{
public static class EnumerableEx
{
public static void ForEach<TSource>(
this IEnumerable<TSource>source, Action<TSource>onNext);
}

public static class ParallelEnumerable
{
public static void ForAll<TSource>(
this ParallelQuery<TSource>source, Action<TSource>action);
}
}
```

ForAll can simultaneously pull results from `ParallelQuery<T>` source with multiple threads, and simultaneously call the specified function on those threads:

```csharp
internal static void ForEachForAll()
{
Enumerable
.Range(0, Environment.ProcessorCount * 2)
.ForEach(value => value.WriteLine()); // 0 1 2 3 4 5 6 7

ParallelEnumerable
.Range(0, Environment.ProcessorCount * 2)
.ForAll(value => value.WriteLine()); // 2 6 4 0 5 3 7 1
}
```

Above is the output after executing the code in a quad core CPU, Unlike ForEach, the values pulled and traced by ForAll is unordered. And if this code runs multiple times, the values can be in different order from time to time. This indeterministic order is the consequence of parallel pulling. The order preservation in parallel query execution is discussed in detail later.

Earlier a WriteLines extension method is defined for `IEnumerable<T>` as a shortcut to call EnumerableEx.ForEach to pull all values and trace them. The following WriteLines overload can be defined for `ParallelQuery<T>` to call ParallelEnumerable.ForAll to simply execute parallel query without calling a function for each query result:

```csharp
public static void WriteLines<TSource>(
this ParallelQuery<TSource> source, Func<TSource, string> messageFactory = null)
{
if (messageFactory == null)
{
source.ForAll(value => Trace.WriteLine(value));
}
else
{
source.ForAll(value => Trace.WriteLine(messageFactory(value)));
}
}
```

## Visualizing parallel query execution

It would be nice if the internal execution of sequential/parallel LINQ queries can be visualized with charts. On Windows, Microsoft provides a tool called Concurrency Visualizer for this purpose. It consists of a library of APIs to trace the execution information, and a Visual Studio extension to render the execution information to chart. This tool is very easy and intuitive. Unfortunately, it only renders chart on Windows along with Visual Studio.

### Using Concurrency Visualizer

To install the Visual Studio extension, just launch Visual Studio, go to Tools => extensions and Updates… => Online, search “Concurrency Visualizer”, and install. Then restart Visual Studio to complete the installation, and go to Analyze => Concurrency Visualizer => Advanced Settings. In the Filter tab, check Sample Events only:

[![clip_image002](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Parallel-LINQ-1_B83B/clip_image002_thumb.jpg "clip_image002")](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Parallel-LINQ-1_B83B/clip_image002_2.jpg)

Then go to Markers tab, check ConcurrencyVisualizer.Markers only:

[![clip_image004](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Parallel-LINQ-1_B83B/clip_image004_thumb.jpg "clip_image004")](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Parallel-LINQ-1_B83B/clip_image004_2.jpg)

In Files tab, specified a proper directory for trace files. Notice the trace files can be very large, which depends on how much information is collected.

[![clip_image006](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Parallel-LINQ-1_B83B/clip_image006_thumb.jpg "clip_image006")](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Parallel-LINQ-1_B83B/clip_image006_2.jpg)

Next, a reference to Concurrency Visualizer library need to be added to project. Microsoft provides this library as a binary on its web page. For convenience, I have created a NuGet package ConcurrencyVisualizer for .NET Framework and .NET Standard. The library provides the following APIs to render timespans on the time line:

```csharp
namespace Microsoft.ConcurrencyVisualizer.Instrumentation
{
public static class Markers
{
public static Span EnterSpan(int category, string text);

public static MarkerSeries CreateMarkerSeries(string markSeriesName);
}

public class MarkerSeries
{
public static Span EnterSpan(int category, string text);
}
}
```

The category parameter is used to determine the color of the rendered timespan, and the text parameter is the label for the rendered timespan.

For Linux and macOS, where Visual Studio is not available, the above Marker, MarkerSeries, and Span types can be manually defined to trace text information:

```csharp
public class Markers
{
public static Span EnterSpan(int category, string spanName) =>
new Span(category, spanName);

public static MarkerSeries CreateMarkerSeries(string markSeriesName) =>
new MarkerSeries(markSeriesName);
}

public class Span : IDisposable
{
private readonly int category;

private readonly string spanName;

private readonly DateTime start;

public Span(int category, string spanName, string markSeriesName = null)
{
this.category = category;
this.spanName = string.IsNullOrEmpty(markSeriesName)
? spanName : $"{markSeriesName}/{spanName}";
this.start = DateTime.Now;
$"{this.start.ToString("o")}: thread id: {Thread.CurrentThread.ManagedThreadId}, category: {this.category}, span: {this.spanName}".WriteLine();
}

public void Dispose()
{
DateTime end = DateTime.Now;
$"{end.ToString("o")}: thread id: {Thread.CurrentThread.ManagedThreadId}, category: {this.category}, span: {this.spanName}, duration: {end – this.start}".WriteLine();
}
}

public class MarkerSeries
{
private readonly string markSeriesName;

public MarkerSeries(string markSeriesName) => this.markSeriesName = markSeriesName;

public Span EnterSpan(int category, string spanName) =>
new Span(category, spanName, this.markSeriesName);
}
```

If a lot of information is traced, more trace listeners can be optionally added to save the information to file or print to console:

```csharp
public partial class Markers
{
static Markers()
{
// Trace to file:
Trace.Listeners.Add(new TextWriterTraceListener(@"D:\Temp\Trace.txt"));
// Trace to console:
Trace.Listeners.Add(new TextWriterTraceListener(Console.Out));
}
}
```

### Visualizing sequential and parallel query execution

Now, the Marker, MarkerSeries, and Span types can be used with LINQ queries and ForEach/ForAll to visualize the sequence/parallel execution on Windows, or trace the execution on Linux and macOS:

```csharp
internal static void RenderForEachForAllSpans()
{
const string SequentialSpan = nameof(EnumerableEx.ForEach);
// Render a timespan for the entire sequential LINQ query execution, with text label "ForEach".
using (Markers.EnterSpan(-1, SequentialSpan))
{
MarkerSeries markerSeries = Markers.CreateMarkerSeries(SequentialSpan);
Enumerable.Range(0, Environment.ProcessorCount * 2).ForEach(value =>
{
// Render a sub timespan for each iteratee execution, with each value as text label.
using (markerSeries.EnterSpan(Thread.CurrentThread.ManagedThreadId, value.ToString()))
{
// Add workload to extend the iteratee execution to a more visible timespan.
for (int i = 0; i < 10_000_000; i++) { }
value.WriteLine(); // 0 1 2 3 4 5 6 7
}
});
}

const string ParallelSpan = nameof(ParallelEnumerable.ForAll);
// Render a timespan for the entire parallel LINQ query execution, with text label "ForAll".
using (Markers.EnterSpan(-2, ParallelSpan))
{
MarkerSeries markerSeries = Markers.CreateMarkerSeries(ParallelSpan);
ParallelEnumerable.Range(0, Environment.ProcessorCount * 2).ForAll(value =>
{
// Render a sub timespan for each iteratee execution, with each value as text label.
using (markerSeries.EnterSpan(Thread.CurrentThread.ManagedThreadId, value.ToString()))
{
// Add workload to extends the iteratee execution to a more visible timespan.
for (int i = 0; i < 10_000_000; i++) { }
value.WriteLine(); // 2 6 4 0 5 3 7 1
}
});
}
}
```

In ForEach and ForAll’s iteratee functions, a for loop of 10 million iterations is executed to add some CPU computing workload to make the function call take longer time, otherwise the rendered timespan of function call can be too small to read. On Windows, click Visual Studio => Analyze => Concurrency Visualizer => Start with Current Project. When the code finishes running, a rich UI is generated. The first tab Utilization shows that the CPU usage was about 25% for a while, which is the sequential LINQ query executing on the quad core CPU. Then the CPU usage became almost 100%, which is the PLINQ execution.

[![clip_image008](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Parallel-LINQ-1_B83B/clip_image008_thumb.gif "clip_image008")](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Parallel-LINQ-1_B83B/clip_image008_2.gif)

The second tab Threads has the chart of timespans. In the thread list on the left, right click the threads not working on LINQ queries and hide them, so that the chart only has the rendered timespans:

[![clip_image010](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Parallel-LINQ-1_B83B/clip_image010_thumb.gif "clip_image010")](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Parallel-LINQ-1_B83B/clip_image010_2.gif)

It uncovers how the LINQ queries execute on this quad core CPU. ForEach query pulls the values and call the specified function sequentially with the main thread. ForAll query does the work with 4 threads (main threads and 3 other worker threads), each thread processed 2 values. The values 6, 0, 4, 2 are processed before 7, 1, 5, 3, which leads to the trace output: 2 6 4 0 5 3 7 1.

Click the ForEach timespan, the Current panel shows the execution duration is 4750 milliseconds. Click ForAll, it shows 1314 milliseconds:

[![clip_image012](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Parallel-LINQ-1_B83B/clip_image012_thumb.gif "clip_image012")](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Parallel-LINQ-1_B83B/clip_image012_2.gif)

This is about 27% of ForEach execution time, which is close to a quarter as expected. It cannot be exactly 25%, because on the device, there are other running processes and threads using CPU, the parallel query also has extra work to manage multithreading, which is covered later in this chapter.

In the last tab Cores, select the LINQ query threads (main thread and other 3 worker thread), and 6760. It shows how the workload is distributed in the 4 cores:

[![clip_image014](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Parallel-LINQ-1_B83B/clip_image014_thumb.gif "clip_image014")](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Parallel-LINQ-1_B83B/clip_image014_2.gif)

Above LINQ visualization code looks noisy, because it mixes the LINQ query code and the visualization code. Following the Single Responsibility Principle, the visualization can be encapsulated for `IEnumerable<T>` and `ParallelQuery<T>`:

```csharp
internal const string ParallelSpan = "Parallel";
internal const string SequentialSpan = "Sequential";

internal static void Visualize<TSource>(
this IEnumerable<TSource> source,
Action<IEnumerable<TSource>, Action<TSource>> query,
Action<TSource> iteratee, string span = SequentialSpan, int category = -1)
{
using (Markers.EnterSpan(category, span))
{
MarkerSeries markerSeries = Markers.CreateMarkerSeries(span);
query(
source,
value =>
{
using (markerSeries.EnterSpan(Thread.CurrentThread.ManagedThreadId, value.ToString()))
{
iteratee(value);
}
});
}
}

internal static void Visualize<TSource>(
this ParallelQuery<TSource> source,
Action<ParallelQuery<TSource>, Action<TSource>> query,
Action<TSource> iteratee, string span = ParallelSpan, int category = -2)
{
using (Markers.EnterSpan(category, span))
{
MarkerSeries markerSeries = Markers.CreateMarkerSeries(span);
query(
source,
value =>
{
using (markerSeries.EnterSpan(Thread.CurrentThread.ManagedThreadId, value.ToString()))
{
iteratee(value);
}
});
}
}
```

And the additional CPU computing workload can also be defined as a function:

```csharp
internal static int ComputingWorkload(int value = 0, int baseIteration = 10_000_000)
{
for (int i = 0; i < baseIteration * (value + 1); i++) { }
return value;
}
```

When it is called as ComputingWorkload() or ComputingWorkload(0), it runs 10 million iterations and output 0; when it is called as ComputingWorkload(1), it runs 20 million iterations and output 1; and so on.

Now the LINQ queries can be visualized in a much cleaner way:

```csharp
internal static void VisualizeForEachForAll()
{
Enumerable
.Range(0, Environment.ProcessorCount * 2)
.Visualize(
EnumerableEx.ForEach,
value => (value + ComputingWorkload()).WriteLine());

ParallelEnumerable
.Range(0, Environment.ProcessorCount * 2)
.Visualize(
ParallelEnumerable.ForAll,
value => (value + ComputingWorkload()).WriteLine());
}
```

### Visualizing chaining queries

Besides visualizing query execution with ForEach and ForAll, the following Visualize overloads can be defined to visualize sequential and parallel queries and render their iteratee function execution as timespans, like Select’s selector, Where’s predicate, etc.:

```csharp
internal static TResult Visualize<TSource, TMiddle, TResult>(
this IEnumerable<TSource> source,
Func<IEnumerable<TSource>, Func<TSource, TMiddle>, TResult> query,
Func<TSource, TMiddle> iteratee,
Func<TSource, string> spanFactory = null,
string span = SequentialSpan)
{
spanFactory = spanFactory ?? (value => value.ToString());
MarkerSeries markerSeries = Markers.CreateMarkerSeries(span);
return query(
source,
value =>
{
using (markerSeries.EnterSpan(
Thread.CurrentThread.ManagedThreadId, spanFactory(value)))
{
return iteratee(value);
}
});
}

internal static TResult Visualize<TSource, TMiddle, TResult>(
this ParallelQuery<TSource> source,
Func<ParallelQuery<TSource>, Func<TSource, TMiddle>, TResult> query,
Func<TSource, TMiddle> iteratee,
Func<TSource, string> spanFactory = null,
string span = ParallelSpan)
{
spanFactory = spanFactory ?? (value => value.ToString());
MarkerSeries markerSeries = Markers.CreateMarkerSeries(span);
return query(
source,
value =>
{
using (markerSeries.EnterSpan(
Thread.CurrentThread.ManagedThreadId, spanFactory(value)))
{
return iteratee(value);
}
});
}
```

Take a simple Where and Select query chaining as example,

```csharp
internal static void VisualizeWhereSelect()
{
Enumerable
.Range(0, 2)
.Visualize(
Enumerable.Where,
value => ComputingWorkload() >= 0, // Where's predicate.
value => $"{nameof(Enumerable.Where)} {value}")
.Visualize(
Enumerable.Select,
value => ComputingWorkload(), // Select's selector.
value => $"{nameof(Enumerable.Select)} {value}")
.WriteLines();

ParallelEnumerable
.Range(0, Environment.ProcessorCount * 2)
.Visualize(
ParallelEnumerable.Where,
value => ComputingWorkload() >= 0, // Where's predicate.
value => $"{nameof(ParallelEnumerable.Where)} {value}")
.Visualize(
ParallelEnumerable.Select,
value => ComputingWorkload(), // Select's selector.
value => $"{nameof(ParallelEnumerable.Select)} {value}")
.WriteLines();
}
```

The sequential and parallel queries are visualized as:

[![clip_image016](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Parallel-LINQ-1_B83B/clip_image016_thumb.gif "clip_image016")](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Parallel-LINQ-1_B83B/clip_image016_2.gif)
