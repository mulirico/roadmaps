# Project organization: answers

## Conceptual

1. A project (`.csproj`) is a single buildable unit that compiles to one output. A
   solution (`.sln`) is a container that groups related projects. A solution with no
   projects builds nothing useful; it only references projects, so there is nothing
   to compile.

2. No. SDK-style projects include all `.cs` files in the project folder and its
   subfolders by default (a glob). You only add items explicitly to exclude files or
   handle special cases.

3. `TargetFramework` says which framework and version the project builds against,
   for example `net8.0`. `netstandard2.0` is a portability target: a library that
   targets it can run on both .NET Framework and modern .NET.

4. `OutputType` of `Exe` produces a runnable application (with an entry point). The
   default output type is a library, which produces a `.dll` with no entry point,
   meant to be referenced by other projects.

5. Top-level statements let you write the body of `Main` directly at the top of a
   file without declaring the `Program` class and `Main` method. Only one file per
   project may use them. The compiler generates the `Program` class and `Main`
   method around those statements, with `args` and an optional integer return as the
   exit code available implicitly.

6. Debug builds have no optimizations and full debug symbols, which makes stepping
   through code reliable; it is the default. Release builds turn on optimizations and
   are meant for deployment. Produce one with `dotnet build -c Release` or
   `dotnet run -c Release`.

7. It adds a `<PackageReference>` entry to the `.csproj` and restores the package.
   The package comes from a NuGet feed, nuget.org by default, into a local cache,
   and is resolved at build time.

8. `dotnet new console`, then `dotnet add package <name>`, then `dotnet run`.

## Coding

9. Example commands:

```bash
dotnet new sln -n MyApp
dotnet new console -o src/App
dotnet new classlib -o src/Core
dotnet sln add src/App src/Core
dotnet add src/App/App.csproj reference src/Core/Core.csproj
```

10. Example:

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>
</Project>
```

11. With implicit usings on, `System` is already imported, so the whole program is:

```csharp
Console.WriteLine("Hello");
```

12. Commands:

```bash
dotnet new console -o JsonDemo
cd JsonDemo
dotnet add package Newtonsoft.Json
```

Code in `Program.cs`:

```csharp
using Newtonsoft.Json;

var payload = new { Name = "Ada", Age = 36 };
string json = JsonConvert.SerializeObject(payload);
Console.WriteLine(json); // {"Name":"Ada","Age":36}
```

Then `dotnet run`.
