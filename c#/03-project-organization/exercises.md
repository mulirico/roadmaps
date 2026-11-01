# Project organization: exercises

Try these without looking at `answers.md`. The coding tasks use the `dotnet` CLI.

## Conceptual

1. What is the difference between a solution (`.sln`) and a project (`.csproj`)?
   Can you build a solution that contains no projects?
2. In an SDK-style project, do you have to list every `.cs` file in the `.csproj`?
   Why or why not?
3. What does the `TargetFramework` element do? Give an example value and say what
   `netstandard2.0` is for.
4. What is the difference between `OutputType` of `Exe` and the default? What gets
   produced in each case?
5. What are top-level statements? How many files in a project may use them, and what
   does the compiler generate from them?
6. What is the difference between a Debug build and a Release build, and how do you
   produce a Release build from the CLI?
7. What does `dotnet add package` change in your project, and where does the package
   come from?
8. Put these in the order you would run them to go from nothing to a running app
   that depends on a NuGet package: `dotnet run`, `dotnet new console`,
   `dotnet add package <name>`.

## Coding

9. Using the CLI, create a solution with a console app and a class library, and add
   a reference from the app to the library. Write the commands you would run.
10. Write a minimal SDK-style `.csproj` for a console app targeting `net8.0` with
    implicit usings and nullable reference types enabled.
11. Rewrite this classic entry point as a top-level statements program:

```csharp
using System;

class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("Hello");
    }
}
```

12. Create a console app, add the `Newtonsoft.Json` package, and write code that
    serializes an anonymous object to JSON and prints it. List the commands and the
    code.
