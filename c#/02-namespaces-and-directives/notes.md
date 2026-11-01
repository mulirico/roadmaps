# Namespaces and directives

## What this covers

How code is grouped into namespaces, how `using` directives bring names into scope,
the newer features (file-scoped namespaces, `global using`, `using static`, aliases,
implicit usings), and the common point of confusion between a `using` directive and
a `using` statement.

## Namespaces

A namespace groups related types and prevents name collisions. The same type name
can exist in two different namespaces without conflict. The fully qualified name is
the namespace path plus the type name, for example `System.Text.StringBuilder`.

Block-scoped declaration:

```csharp
namespace MyApp.Services
{
    public class OrderService { }
}
```

File-scoped declaration (C# 10 and later), one namespace for the whole file, less
indentation:

```csharp
namespace MyApp.Services;

public class OrderService { }
```

A file can hold only one file-scoped namespace, and it must come before any type.

## using directive

A `using` directive brings the names of a namespace into scope so you can write
`StringBuilder` instead of `System.Text.StringBuilder`.

```csharp
using System.Text;

var sb = new StringBuilder();
```

Variants:

- `using static System.Math;` imports static members, so you can write `Sqrt(2)`
  instead of `Math.Sqrt(2)`.
- `using Json = System.Text.Json;` creates an alias for a namespace or type, useful
  for resolving name collisions or shortening a long name.
- `global using System;` (C# 10 and later) applies the directive to every file in
  the project. Put these in one file, often `GlobalUsings.cs`.

## Implicit usings

SDK-style projects can enable implicit usings with
`<ImplicitUsings>enable</ImplicitUsings>` in the `.csproj`. The SDK then adds a set
of common `global using` directives for you (for example `System`,
`System.Collections.Generic`, `System.Linq`), which is why a new console app can
call `Console.WriteLine` without writing `using System;`.

## using directive vs using statement

These share the keyword but are unrelated.

- A `using` directive appears at the top of a file (or as `global using`) and
  imports names. It has no runtime effect.
- A `using` statement (or declaration) manages the lifetime of an
  `IDisposable` object and calls `Dispose()` when the scope ends. It is about
  releasing resources, not importing names.

```csharp
using var reader = new StreamReader("file.txt"); // using declaration
string line = reader.ReadLine();
// reader.Dispose() is called at the end of the enclosing scope
```

## Name collisions and fully qualified names

When two imported namespaces both define `Timer`, the compiler reports an ambiguous
reference. Resolve it with an alias, or by using the fully qualified name such as
`System.Threading.Timer`. The `global::` prefix forces resolution from the root
namespace, for example `global::System.Console`.

## Documentation

- Namespaces: https://learn.microsoft.com/dotnet/csharp/fundamentals/types/namespaces
- using directive: https://learn.microsoft.com/dotnet/csharp/language-reference/keywords/using-directive
- File-scoped namespace declaration: https://learn.microsoft.com/dotnet/csharp/language-reference/keywords/namespace
- Implicit usings: https://learn.microsoft.com/dotnet/core/project-sdk/overview#implicit-using-directives
- using statement: https://learn.microsoft.com/dotnet/csharp/language-reference/statements/using
