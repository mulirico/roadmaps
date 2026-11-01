# Organization of a C# .NET project

## What this covers

How a .NET codebase is structured: the difference between a solution and a project,
what the `.csproj` file holds, the `dotnet` CLI commands you use daily, how
dependencies work through NuGet, the entry point and top-level statements, and the
Debug and Release configurations.

## Solution vs project

- A project (`.csproj`) is a single buildable unit. It compiles to one output, for
  example a console app (`.exe`/`.dll`), a class library (`.dll`), or a web app.
- A solution (`.sln`) is a container that groups one or more related projects so you
  can build and open them together. A solution builds nothing on its own; it
  references projects.

A common layout is one solution with an app project, a class library for the domain
logic, and a test project.

## SDK-style .csproj

Modern projects use the SDK-style format, which is short and declarative. A minimal
console project:

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

Key elements:

- `Sdk` selects the SDK. `Microsoft.NET.Sdk` for console and library,
  `Microsoft.NET.Sdk.Web` for web apps.
- `TargetFramework` sets the framework the project builds against, for example
  `net8.0`.
- `OutputType` of `Exe` produces an executable; the default (library) produces a
  `.dll`.
- `Nullable` enables nullable reference type analysis.
- SDK-style projects include all `.cs` files in the folder by default, so you do not
  list source files one by one.

## Target framework

The target framework moniker (TFM) says which framework and version you build for,
for example `net8.0`. `netstandard2.0` is a portability target for libraries that
need to run on both .NET Framework and modern .NET. A project can target multiple
frameworks with `<TargetFrameworks>` (plural).

## dotnet CLI

The commands you use most:

- `dotnet new <template>` creates a project, for example `dotnet new console`,
  `dotnet new classlib`, `dotnet new xunit`, `dotnet new sln`.
- `dotnet build` compiles.
- `dotnet run` builds and runs a project.
- `dotnet test` runs tests.
- `dotnet add package <name>` adds a NuGet dependency.
- `dotnet add reference <path>` adds a project-to-project reference.
- `dotnet sln add <path>` adds a project to a solution.
- `dotnet restore` restores dependencies (build and run do this automatically).

## Dependencies and NuGet

NuGet is the .NET package manager. `dotnet add package Newtonsoft.Json` records the
dependency as a `<PackageReference>` in the `.csproj`. Packages come from a feed
(nuget.org by default). Dependencies restore into a local cache and are resolved at
build time.

## Entry point and top-level statements

A program starts at the `Main` method:

```csharp
class Program
{
    static void Main(string[] args) { }
}
```

Since C# 9, a project can use top-level statements: you write the body of `Main`
directly in one file, and the compiler generates the `Main` method for you. A new
console app looks like this:

```csharp
Console.WriteLine("Hello");
```

Only one file in the project may contain top-level statements. `args` and an
optional `return` for the exit code are available implicitly.

## Debug vs Release

Build configurations change how the code is compiled.

- Debug: no optimizations, full debug symbols, easier to step through. The default
  for `dotnet build` and `dotnet run`.
- Release: optimizations on, intended for deployment. Build it with
  `dotnet build -c Release` or `dotnet run -c Release`.

## Common folder conventions

There is no single enforced layout, but a frequent convention is `src/` for projects
and `tests/` for test projects, each project in its own folder with its `.csproj`,
and the `.sln` at the repository root.

## Documentation

- .NET project SDK overview: https://learn.microsoft.com/dotnet/core/project-sdk/overview
- .NET CLI (dotnet command): https://learn.microsoft.com/dotnet/core/tools/
- Target frameworks: https://learn.microsoft.com/dotnet/standard/frameworks
- Top-level statements: https://learn.microsoft.com/dotnet/csharp/fundamentals/program-structure/top-level-statements
- Manage dependencies with NuGet: https://learn.microsoft.com/nuget/consume-packages/install-use-packages-dotnet-cli
- Solutions in .NET: https://learn.microsoft.com/visualstudio/ide/solutions-and-projects-in-visual-studio
