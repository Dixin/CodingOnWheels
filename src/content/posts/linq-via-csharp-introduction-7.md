---
title: "Functional Programming and LINQ Paradigm (1) Getting Started with .NET/Core, C# and LINQ"
published: 2018-05-28
description: "This is a tutorial of functional programming and LINQ in C# language. The contents was initially based on my . Hope it helps."
image: ""
tags: [".NET", ".NET Core", ".NET Standard", "C#", "LINQ"]
category: ".NET"
draft: false
lang: ""
---

> [!TIP]  
> [Functional Programming and LINQ via C#](/posts/linq-via-csharp) Series

## Latest version: [https://CodingOnWheels.com/posts/linq-via-csharp-introduction](/posts/linq-via-csharp-introduction "https://CodingOnWheels.com/posts/linq-via-csharp-introduction")

This is a tutorial of functional programming and LINQ in C# language. The contents was initially based on my [LINQ via C# talks](/posts/linq-via-csharp-events-posters-design). Hope it helps.

[![LINQ-via-CSharp_thumb](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Functional-Programmin.NETCore-C-and-LINQ_14FDE/LINQ-via-CSharp_thumb_thumb.png "LINQ-via-CSharp_thumb")](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Functional-Programmin.NETCore-C-and-LINQ_14FDE/LINQ-via-CSharp_thumb_2.png)

## Cross platform .NET, C# and LINQ

In 2002, C# was initially introduced with .NET Framework on Windows. Since then, many functional features including LINQ has been built into C# language and .NET Framework. There are also many other frameworks joins the .NET family, which enable C# and LINQ to work on many platforms.

### .NET Framework

Microsoft [.NET Framework](https://en.wikipedia.org/wiki/.NET_Framework) (pronounced “dot net”) is a free development framework on Windows, widely used to build applications and services with simple programming model and great productivity. .NET Framework is based on [Common Intermediate Language](https://en.wikipedia.org/wiki/Common_Intermediate_Language) (CIL), and consists of [Common Language Runtime](https://en.wikipedia.org/wiki/Common_Language_Runtime) (CLR), [Framework Class Library](https://en.wikipedia.org/wiki/Framework_Class_Library) (FCL):

-   CIL is the object-oriented assembly language used by .NET Framework.
-   FCL is a set of built-in libraries of rich APIs implemented as classes, interfaces, and structures, etc. It is the fundamental used by .NET applications and services to access system functionality. FCL provides primitive types, exceptions, collections, I/O, threading, reflection, text processing, database access, and LINQ, etc.
-   CLR is the runtime environment that works like a virtual machine. All .NET applications and services are executed by CLR. CLR provides features including [automatic memory management](https://msdn.microsoft.com/en-us/library/f144e03t.aspx), thread management, structured exception handling, type safety, security, just-in-time (JIT) compiler which compiles CIL to machine code, etc.

[C#](https://msdn.microsoft.com/en-us/library/ms228593.aspx) language (pronounced “c sharp”) is a general purpose high level language, and standardized by ECMA-334 and ISO/IEC 23270 standards. Microsoft’s C# compiler is an implementation of these standards. It compiles C# to CIL, which can be executed by CLR. C# is type-safe, generic, object-oriented and functional programming language. It is modern, expressive and productive. There are also [other high level languages](https://en.wikipedia.org/wiki/List_of_CLI_languages) that can be used to build .NET applications and services, like VB.NET, F#, etc., which are compiled or interpreted to CIL as well. C# is the most popular .NET language used by millions of people. Microsoft provides [Visual Studio,](https://en.wikipedia.org/wiki/Microsoft_Visual_Studio) a powerful integrated development environment (IDE), with built-in support for .NET and C# software development.

The real world applications and services work with data, which can be of any form, like data objects in local memory, data in XML format, data stored with database, etc. Traditionally, a specific programming model is required to work with each kind of data source. For example, traditionally, querying a sequence of data objects in local memory can be quite different from querying data rows from a table in database. For .NET and C# programming, Microsoft provides a general purpose solution applies to many data sources, that is LINQ. When searching “LINQ” with [Bing](https://www.bing.com/search?q=linq) or [Google](https://www.google.com/search?q=linq), the top item on the first result page is an ad of the [LINQ hotel & casino](https://www.caesars.com/linq) in Las Vegas:

[![8753178_25_z_thumb](https://aspblogs.z22.web.core.windows.net/dixin/Windows-Live-Writer/Introducing-LINQ-1-What-Is-LINQ_10D64/8753178_25_z_thumb_thumb.jpg "8753178_25_z_thumb")](https://aspblogs.z22.web.core.windows.net/dixin/Windows-Live-Writer/Introducing-LINQ-1-What-Is-LINQ_10D64/8753178_25_z_thumb_2.jpg)

However, in this tutorial, LINQ stands for something more serious, “Language-INtegrated Query” (pronounced “link”). It is a set of general purpose data query features enabling a simple, consistent, and powerful bridge between the programming domain and many different data domains. LINQ consists of language features and .NET FCL features:

-   Native .NET languages features are added for data query capabilities. For C#, language features, including lambda expression, query expression, etc., are added to compose declarative and functional data queries.
-   Data access APIs are implemented in .NET FCL, including interfaces and classes representing the data sources, query methods implementing the query logic, etc.

For .NET applications and services using LINQ, at compile time, the data queries in native languages are compiled to regular API calls; At runtime, CLR executes these API calls to query the data sources. Microsoft implements LINQ syntaxes for languages including C#, VB.NET, F#, etc., and also implements LINQ APIs in FCL to work with CLR objects, XML data, and database. The language features can work FCL APIs as well as custom APIs, which enables LINQ to work with many data sources.

LINQ is rooted in Microsoft's [Cω](http://en.wikipedia.org/wiki/C%CF%89) research project, and was first released as a part of [.NET Framework 3.5](http://en.wikipedia.org/wiki/.NET_Framework_3.5) and C# 3.0. The following table shows the position of LINQ in the history roadmap of .NET Framework and C# language:

| Year | Visual Studio | .NET Framework | Framework features                                                              | CLR           | C#  |
|------|---------------|----------------|---------------------------------------------------------------------------------|---------------|-----|
| 2002 | .NET 2002     | 1.0            | CLR, FCL (ADO.NET, ASP.NET, etc.)                                               | 1.0           | 1.0 |
| 2003 | .NET 2003     | 1.1            | IPv6, Oracle database, etc.                                                     | 1.1           | 1.1 |
| 2003 |               |                |                                                                                 |               | 1.2 |
| 2005 | 2005          | 2.0            | Generics, full 64 bit computing, etc.                                           | 2.0           | 2.0 |
| 2006 |               | 3.0            | WCF, WPF, WF, etc.                                                              |               |     |
| 2007 | 2008          | 3.5            | LINQ, etc.                                                                      |               | 3.0 |
| 2010 | 2010          | 4.0            | TPL, Parallel LINQ, etc.                                                        | 4 (not “4.0”) | 4.0 |
| 2012 | 2012          | 4.5            | Zip, Parallel LINQ improvement, etc.                                            |               | 5.0 |
| 2013 | 2013          | 4.5.1          | Automatic binding redirection, etc.                                             |               |     |
| 2014 |               | 4.5.2          | New ASP.NET APIs, etc.                                                          |               |     |
| 2015 | 2015          | 4.6            | New 64-bit JIT compiler, etc.                                                   |               | 6.0 |
| 2015 |               | 4.6.1          | Cryptography improvement, .NET Standard 2.0 support with additional files, etc. |               |     |
| 2016 |               | 4.6.2          | SQL Server client improvement, etc.                                             |               |     |
| 2017 | 2017          |                |                                                                                 |               | 7.0 |
| 2017 |               | 4.7            | Azure SQL Database connectivity improvement, etc.                               |               |     |
| 2017 |               |                |                                                                                 |               | 7.1 |
| 2017 |               | 4.7.1          | Built-in .NET Standard 2.0 support, etc.                                        |               |     |
| 2017 |               |                |                                                                                 |               | 7.2 |

### .NET Core, UWP, Mono, Xamarin and Unity

After 15+ years, .NET Framework has been a rich ecosystem on Windows. Besides .NET Framework, C# also works on many other frameworks and platforms. In 2016, Microsoft released .NET Core, a free, open source and cross-platform version of .NET Framework. .NET Core is essentially a fork a .NET Framework. it is still based on CIL, with a runtime called CoreCLR, and class libraries called CoreFX. The same C# language works with .NET Core, as well as fore mentioned F# and VB.NET. As the name suggests, .NET Core implements the core features of .NET Framework. So it can be viewed as a subset of .NET Framework. It is designed to be a lightweight and high performance framework to build applications and services on Windows, macOS, and many Linux distributions, including Read Hat, Ubuntu, CentOS, Debian, Fedora, OpenSUSE, Oracle Linux, etc., so that it works on a wide range of devices, clouds, and embedded/IoT scenarios. The following table shows .NET Core is released in a more agile iteration:

| Year     | .NET Core         | .Features                                        |
|----------|-------------------|--------------------------------------------------|
| Jun 2016 | 1.0               | CoreCLR, CoreFX, WCF, ASP.NET Core, etc.         |
| Sep 2016 | 1.0.1             | Update for 1.0.                                  |
| Oct 2016 | 1.0.2             | Update for 1.0.                                  |
| Nov 2016 | 1.1               | More APIs, performance improvements, etc.        |
| Dec 2016 | 1.0.3             | Update for 1.0.                                  |
| Mar 2017 | 1.0.4/1.1.1       | Update for 1.0/1.1.                              |
| May 2017 | 1.0.5/1.1.2       | Update for 1.0/1.1.                              |
| Aug 2017 | 2.0               | .NET Standard 2.0, performance improvement, etc. |
| Sep 2017 | 1.0.6/1.1.3       | Update for 1.0/1.1.                              |
| Nov 2017 | 1.0.7/1.1.4       | Update for 1.0/1.1.                              |
| Nov 2017 | 1.0.8/1.1.5/2.0.3 | Update for 1.0/1.1/2.0.                          |
| Dec 2017 | 2.0.4             | Update for 2.0.                                  |
| Jan 2018 | 1.0.9/1.1.6/2.0.5 | Update for 1.0/1.1/2.0.                          |

Microsoft also released Universal Windows Platform (UWP), the app model for Windows 10. UWP enables C# (as well as VB.NET, C++, JavaScript) to develop Microsoft Store application that can work cross all Windows 10 device families, including PC, tablet, phone, Xbox, HoloLens, Surface Hub, IoT, etc. [UWP takes advantage of .NET Core](https://msdn.microsoft.com/en-us/magazine/mt814993.aspx). In Debug mode, UWP app is compiled to CIL, and runs against CoreCLR. In Release mode, UWP app is compiled to native binaries for better performance, and runs against [.NET Native](https://blogs.windows.com/buildingapps/2015/08/20/net-native-what-it-means-for-universal-windows-platform-uwp-developers/) runtime.

Besides .NET Core and UWP, Mono (Monkey in Spanish) is another open source implementation of .NET Framework based on the ECMA standards for C# and CLR. Mono was initially released in 2004. It works cross many platforms, including Windows, macOS, most Linux distributions, BSD, Solaris, Android, iOS, and game consoles like Xbox, PlayStation, Wii, etc.. Based on Mono, Xamarin is a framework for building native mobile apps on Windows, Android and iOS with C#. Microsoft acquired Xamarin in 2016 and has made it open source and available as free.

C# is also the language for Unity, a [cross platform](https://docs.unity3d.com/Manual/PlatformDependentCompilation.html) game engine developed by Unity Technologies. Unity also [takes advantage of Mono](https://msdn.microsoft.com/en-us/magazine/dn759441.aspx) to enable C# to develop games for Windows, macOS, Linux, Android, iOS, and game consoles like Xbox, PlayStation, Wii, etc. Unity used to support UnityScript, a JavaScript-like language, and Boo language. Now UnityScript and Boo are being [deprecated](https://blogs.unity3d.com/2017/08/11/unityscripts-long-ride-off-into-the-sunset/) regarding the popularity of C#.

The following table summarizes these framework's languages, base API surface, runtime for managed code, supported application models, and supported platforms:

||.NET Framework|.NET Core|UWP|Xamarin|Unity|
|---|---|---|---|---|---|
|Languages|C#, VB.NET, F#, etc.|C#, F#, VB.NET|C#, VB.NET, C++, JavaScript|C#|C#, UnityScript (deprecated), Boo (deprecated)|
|Base API surface|.NET FCL|CoreFX|Universal device family APIs|Mono base libraries|Mono base libraries|
|Managed runtime|CLR|CoreCLR|.NET Native runtime|Mono runtime|Mono runtime|
|Application models|Windows desktop applications and services|Cross-platform services|Microsoft Store apps|Mobile apps|Games|
|Platforms|Windows|Windows, macOS, Linux|Windows|Windows, Android, iOS|Windows, macOS, Linux, Android, iOS, game consoles|

### .NET Standard

The same C# language works on many frameworks and platforms. However, each framework provides its own base API surface for C# developers. To prevent APIs’ fragmentation, provide a unified development experience, and enable better code sharing, Microsoft defines .NET Standard specification. .NET Standard is a list of APIs, which is the base API surface should be implemented by any framework in the .NET family. .NET Standard is represented by NuGet package NETStandard.Library, which has a reference assembly netstandard.dll. The latest major release of .NET Standard is 2.0. It has 32k+ APIs. It is [supported](https://github.com/dotnet/standard/blob/master/docs/versions.md) by:

-   .NET Framework 4.6.1/4.6.2/4.7 (support with additional files), .NET Framework 4.7.1 (built-in support)
-   .NET Core 2.0
-   Mono 5.4
-   UWP 10.0.16299
-   Xamarin.Forms 2.4, Xamarin.Mac 3.8, Xamarin.Android 8.0, Xamarin.iOS 10.14
-   Unity 2018

[![image_thumb](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Functional-Programmin.NETCore-C-and-LINQ_14FDE/image_thumb_thumb.png "image_thumb")](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Functional-Programmin.NETCore-C-and-LINQ_14FDE/image_thumb_2.png)

This standardization provides great consistency and productivity for C# developers – one language and one set of base APIs can be used to develop many kinds of applications working cross many platforms. In the perspective of C# developer, the development experience becomes to use one lanuage and one set of base APIs to develop many kinds of applications and servers on many platforms:

||.NET Framework|.NET Core|UWP|Xamarin|Unity|
|---|---|---|---|---|---|
|Language|C#|C#|C#|C#|C#|
|Base API surface|.NET Standard|.NET Standard|.NET Standard|.NET Standard|.NET Standard|
|Application models|Windows desktop applications and services|Cross-platform services|Microsoft Store apps|Mobile apps|Games|
|Platforms|Windows|Windows, macOS, Linux|Windows|Windows, Android, iOS|Windows, macOS, Linux, Android, iOS, game consoles|

The LINQ language features are part of the C# language standard, and the LINQ APIs are part of the .NET Standard, so LINQ is available on all frameworks in the .NET family, with one set of language syntax and one set of APIs. This tutorial covers the cross platform C# language and cross-platform LINQ technologies provided by Microsoft and adopting to .NET Standard 2.0, including LINQ to Objects, Parallel LINQ, LINQ to XML, LINQ to Entities.

### C# functional programming

.NET Standard is an object-oriented collection of reusable types, CIL is a object-oriented assembly language, and C# is also initially an object-oriented programming language, fully supporting encapsulation, inheritance, and polymorphism, so that .NET APIs and C# language work together seamlessly. In the meanwhile, C# also supports functional programming. As a typical example, LINQ is extensively functional. In C#, functions are first class citizens just like objects are. C# has plenty of functional features, like closure, higher-order function, anonymous function, etc. The LINQ features, like query expressions, lambda expression, etc., are also functional features instead of object-oriented features.

Functional programming is different from object-oriented programming in many aspects. Functional programming is usually more self-contained, more stateless, more immutable, more lazy, more side effects management, etc. The most intuitive difference is, functional programming is more declarative instead of imperative. It focus on describing what to do, instead of specifying the execution details of how to do. As a result, functional programming can be very expressive and productive. When working with data, as a typical example, functional LINQ queries provide the general capabilities of describing what is the query logic for different data source, rather than specifying the execution details of how to access and query each specific data source, so that LINQ can be one powerful language to work with many data sources. Functional programming can also be more scalable. For example, when working with data using LINQ, it can be very easy to parallelize the workload multiple processor cores.

In C# development, object-oriented programming and functional programming live in harmony. For example, when a functional LINQ query works with data in local memory, the LINQ query actually works with CLR objects which represent the data. Also, when a LINQ query is executed, LINQ APIs are called, and the LINQ APIs can be internally implemented with imperative object-oriented programming.

## This tutorial

This tutorial discusses cross-platform functional programming and LINQ programming via the latest C# 7.0 language, from real world development to underlying theories. It covers both .NET Framework (for Windows) and .NET Core (for Windows, macOS and Linux). This entire tutorial is based on the latest language and frameworks. It covers C#’s functional features and functional programming aspects, and the detailed usage and internal mechanisms of mainstream LINQ technologies for different data domains, including LINQ to Objects, Parallel LINQ, LINQ to XML, and LINQ to Entities. It also demystifies the underlying quintessential theories of functional programming and LINQ, including Lambda Calculus and Category Theory.

As an in-depth tutorial, some basic understanding of programming and C# is necessary. The target audiences are those who want to learn C# functional programming for Windows development and cross-platform development, and those who want to learn how to use LINQ in C# to work with data in applications and services. This tutorial is also for advanced audiences who want to learn the quintessence of functional programming to build a deep and general understanding, and those who want to learn internal details of LINQ in order to build custom LINQ APIs or providers.

The contents are organized as the following chapters:

-   Part 1 Code - covers functional programming via C#, and fundamentals of LINQ.
    -   Chapter 1 Functional programming and LINQ paradigm
        -   What is LINQ, how LINQ uses language to work with many different data domains.
        -   Programming paradigm, imperative vs. declarative programming, object-oriented vs. functional programming.
    -   Chapter 2 C# functional programming in-depth
        -   C# fundamentals for beginners.
        -   Aspects of functional programming via C#, including function type, named/anonymous/local function, closure, lambda, higher-order function, currying, partial application, first class function, function composition, query expression, covariance/contravariance, immutability, tuple, purity, async function, pattern matching, etc., including how C# is processed at compile time and runtime.
-   Part 2 Data - covers how to use functional LINQ to work with different data domains in the real world, and how LINQ works internally.
    -   Chapter 3 LINQ to Objects
        -   How to use functional LINQ queries to work with objects, covering all LINQ and Ix.
        -   How the LINQ to Objects query methods are implemented, how to implement useful custom LINQ queries.
    -   Chapter 4 LINQ to XML
        -   How to modeling XML data, and use functional LINQ queries to work with XML data.
        -   How to use the other LINQ to XML APIs to manipulate XML data.
    -   Chapter 5 Parallel LINQ
        -   How to use parallelized functional LINQ queries to work with objects.
        -   Performance analysis for parallel/sequential LINQ queries.
    -   Chapter 6 Entity Framework/Core and LINQ to Entities
        -   How to model database with object-relational mapping, and use functional LINQ queries to work with relational data in database.
        -   How the C# LINQ to Entities queries are implemented to work with database.
        -   How to change data in database, and handle concurrent conflicts.
        -   Performance tips and asynchrony.
-   Part 3 Theories - demystifies the abstract mathematics theories, which are the rationale and foundations of LINQ and functional programming.
    -   Chapter 7 Lambda Calculus via C#
        -   Core concepts of lambda calculus, bound and free variables, reduction (α-conversion, β-reduction, η-conversion), etc.
        -   How to use lambda functions to represent values, data structures and computation, including Church Boolean, Church numbers, Church pair, Church list, and their operations.
        -   Combinators and combinatory logic, including SKI combinator calculus, fixed point combinator for function recursion, etc.
    -   Chapter 8 Category Theory via C#
        -   Core concepts of category theory, including category, object, morphism, monoid, functor, natural transformation, applicative functor, monad, and their laws.
        -   How these concepts are applied in functional programming and LINQ.
        -   How to manage I/O, state, exception handling, shared environment, logging, and continuation, etc., in functional programming.

This tutorial delivers highly reusable knowledge:

-   It covers C# knowledge in detail, which can be generally used in any programming paradigms other than functional programming.
-   It is a cross platform tutorial, covering both .NET Framework for Windows and .NET Core for Windows, macOS, Linux
-   It delivers LINQ usage and implementation for mainstream data domains, which also enables developer to use the LINQ technologies for other data domains, or build custom LINQ APIs for specific data scenarios.
-   It also demystifies the abstract mathematics knowledge for functional programming, which applies to all functional languages, so it greatly helps understanding any other functional languages too.

## Code examples

All code examples are available on GitHub: [https://github.com/Dixin/Tutorial](https://github.com/Dixin/Tutorial "https://github.com/Dixin/Tutorial"). If there is any issue, please feel free to file it here: [https://github.com/Dixin/Tutorial/issues/new](https://github.com/Dixin/Tutorial/issues/new "https://github.com/Dixin/Tutorial/issues/new").

To save the space and paper, all code examples in this tutorial omit argument null check.

## Author

I have been a developer for 12 years. I was a Software Development Engineer in Microsoft during 2010 - 2016. Before I join Microsoft, I was a C# MVP.

I have a physics degree, and I learnt computer science by myself, so I understand it is not so that easy. In this tutorial, I try to discuss C#, LINQ, functional programming with simple words and intuitive examples.

## Start coding

All tools, libraries, services involved in this tutorial are either free, or with free option available. In theory, any text editor can be used for C# programming, but a power tools can greatly improve the productivity. The following are the free tools provided by Microsoft:

-   [Visual Studio Community Edition](https://www.visualstudio.com/en-us/products/vs-2015-product-editions.aspx): the free and fully featured Visual Studio for Windows, the powerful and productive flagship integrated development environment (IDE) for C#/.NET and other development.
-   [Visual Studio Code](https://code.visualstudio.com/): the free and rich code editor for Windows, macOS and Linux, supporting coding of C# and other languages with extensions.
-   [Visual Studio for Mac](https://docs.microsoft.com/en-us/visualstudio/mac/): the free and sophisticated IDE for macOS, supporting development of .NET Core, Xamarin, etc.

### Start coding with Visual Studio (Windows)

The free Community Edition of Visual Studio can be downloaded from the Microsoft official website: [https://visualstudio.com](https://visualstudio.com/ "https://www.visualstudio.com"). To start C# programming with .NET Core, select the “.NET Core cross-platform development” workload; To start C# programming with .NET Framework on Windows, select the “.NET desktop development” workload:

[![image_thumb3](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Functional-Programmin.NETCore-C-and-LINQ_14FDE/image_thumb3_thumb.png "image_thumb3")](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Functional-Programmin.NETCore-C-and-LINQ_14FDE/image_thumb3_2.png)

[![image_thumb2](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Functional-Programmin.NETCore-C-and-LINQ_14FDE/image_thumb2_thumb.png "image_thumb2")](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Functional-Programmin.NETCore-C-and-LINQ_14FDE/image_thumb2_2.png)

This installs Visual Studio along with .NET Framework SDK/.NET Core SDK. To install the latest version of .NET Framework SDK/.NET Core SDK, follow the steps from the Microsoft official website: [https://dot.net](https://dot.net/). After all installation is done, launch Visual Studio. For .NET Core, click File => New => Project to create a new console application:

[![image_thumb7](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Functional-Programmin.NETCore-C-and-LINQ_14FDE/image_thumb7_thumb.png "image_thumb7")](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Functional-Programmin.NETCore-C-and-LINQ_14FDE/image_thumb7_2.png)

In Solution Explorer, under this application, there is a Program.cs file, which has the application’s entry point Main:.

```csharp
using System;

namespace ConsoleApp
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello World!");
        }
    }
}
```

Then right click the project, click Properties. In the project property window, go to the Build tab, click the Advanced button, and change the language version to latest:

[![image_thumb9](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Functional-Programmin.NETCore-C-and-LINQ_14FDE/image_thumb9_thumb.png "image_thumb9")](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Functional-Programmin.NETCore-C-and-LINQ_14FDE/image_thumb9_2.png)

Now right click the project again, click “Manage NuGet Packages” to install the NuGet packages used in this tutorial:

-   FSharp.Core
-   linqtotwitter
-   Microsoft.Azure.DocumentDB.Core
-   Microsoft.EntityFrameworkCore.SqlServer
-   Microsoft.Extensions.Configuration.Json
-   Mono.Cecil
-   System.Interactive
-   System.Memory
-   System.Reflection.Emit.Lightweight
-   System.Threading.Tasks.Extensions

[![image_thumb4](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Functional-Programmin.NETCore-C-and-LINQ_14FDE/image_thumb4_thumb.png "image_thumb4")](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Functional-Programmin.NETCore-C-and-LINQ_14FDE/image_thumb4_2.png)

For .NET Framework, create a console application of Windows classic desktop:

[![image_thumb8](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Functional-Programmin.NETCore-C-and-LINQ_14FDE/image_thumb8_thumb.png "image_thumb8")](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Functional-Programmin.NETCore-C-and-LINQ_14FDE/image_thumb8_2.png)

Change the language version to latest as well, and install the following packages:

-   ConcurrencyVisualizer
-   EntityFramework
-   FSharp.Core
-   linqtotwitter
-   Microsoft.Azure.DocumentDB
-   Microsoft.TeamFoundationServer.ExtendedClient
-   Mono.Cecil
-   System.Collections.Immutable
-   System.Interactive
-   System.Memory
-   System.Threading.Tasks.Extensions

Then right click the created project’s References child node, click Add Reference…, add the following framework assemblies:

-   System.Configuration
-   System.Transactions

This Parallel LINQ chapter also uses a free Visual Studio extensions for .NET Framework, Concurrent Visualizer provided by Microsoft. it can be installed from Tools => Extensions and Updates….

[![image_thumb1](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Functional-Programmin.NETCore-C-and-LINQ_14FDE/image_thumb1_thumb.png "image_thumb1")](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Functional-Programmin.NETCore-C-and-LINQ_14FDE/image_thumb1_2.png)

More code files can be added under the application. Now press F5 to build, run and debug the application in Visual Studio.

### Start coding with Visual Studio Code (Windows, macOS and Linux)

The free Visual Studio Code can be downloaded and installed from Microsoft official website: [https://code.visualstudio.com](https://code.visualstudio.com/). This tutorial also uses 2 extensions for Visual Studio Code: C# extension for C# programming, and mssql extension for SQL execution in the LINQ to Entities chapter. These extensions are both provided by Microsoft.

[![image_thumb5](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Functional-Programmin.NETCore-C-and-LINQ_14FDE/image_thumb5_thumb.png "image_thumb5")](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Functional-Programmin.NETCore-C-and-LINQ_14FDE/image_thumb5_2.png)

The .NET Core SDK needs to be installed separately, by following the steps from Microsoft official website: [https://dot.net](https://dot.net/ "https://dot.net"). The installation can be verified by the `dotnet –version` command, which outputs the version of .NET Core SDK. To start coding, create a directory for a new console application, then go to this directory, run `dotnet new console`. 2 files are created, Program.cs and ConsoleApp.csproj. Program.cs is the C# code file, which is the same as above Program.cs created by Visual Studio. ConsoleApp.csproj is the project file containing the metadata and build information for this console application.

The NuGet packages used by this tutorial can be added with the `dotnet add package {package name}` command. For the packages only available as preview, the version has to be specified: `dotnet add package {package name} –version {version}`.

From this directory, run code . command to start Visual Studio Code. Visual Studio Code should prompt “Required assets to build and debug are missing from ‘ConsoleApp’. Add them?”. Click Yes, Visual Studio Code should create the debug configuration files in a .vscode subdirectory. Now, press F5 to build, run and debug the application in Visual Studio Code.

### Start coding with Visual Studio for Mac (macOS)

The free Visual Studio for Mac can be downloaded and installed from Microsoft official website: [https://www.visualstudio.com/vs/visual-studio-mac](https://www.visualstudio.com/vs/visual-studio-mac "https://www.visualstudio.com/vs/visual-studio-mac"). Then launch Visual Studio for Mac, click New Project button on the welcome page to create a new .NET Core console application:

[![image_thumb12](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Functional-Programmin.NETCore-C-and-LINQ_14FDE/image_thumb12_thumb.png "image_thumb12")](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Functional-Programmin.NETCore-C-and-LINQ_14FDE/image_thumb12_2.png)

Then right click the created project, click Options. In the opened project options window, click the General tab under Build, change the language version to latest:

[![image_thumb11](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Functional-Programmin.NETCore-C-and-LINQ_14FDE/image_thumb11_thumb.png "image_thumb11")](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Functional-Programmin.NETCore-C-and-LINQ_14FDE/image_thumb11_2.png)

Then right click the created project’s Dependencies child node, click Add Packages, install the fore mentioned NuGet packages:

[![image_thumb13](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Functional-Programmin.NETCore-C-and-LINQ_14FDE/image_thumb13_thumb.png "image_thumb13")](https://aspblogs.z22.web.core.windows.net/dixin/Open-Live-Writer/Functional-Programmin.NETCore-C-and-LINQ_14FDE/image_thumb13_2.png)

Now, just press F5 to build, run and debug the code in Visual Studio for Mac.
