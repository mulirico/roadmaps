# C# and .NET study notes

Personal study material for C# and the .NET environment, focused on basic concepts
and the kind of questions asked in interviews for mid-level software engineer roles.

Each topic is a folder with three files:

- `notes.md`: a short summary of the concepts and links to the official Microsoft
  documentation.
- `exercises.md`: practice questions and small coding tasks.
- `answers.md`: the answer key for that topic, kept separate so you can test
  yourself first.

## Topics

1. [Basic types](01-basic-types/notes.md)
2. [Namespaces and directives](02-namespaces-and-directives/notes.md)
3. [Project organization](03-project-organization/notes.md)
4. [Types: value vs reference](04-types-value-vs-reference/notes.md)

## Suggested study flow

1. Read `notes.md` for the topic.
2. Open the documentation links and read the parts that are new to you.
3. Do the exercises in `exercises.md` without looking at the answers. Write code and
   run it where the exercise asks for it.
4. Check your work against `answers.md` and review anything you missed.

## How to run C# snippets

Most exercises can be tried in a throwaway console project:

```bash
dotnet new console -o scratch
cd scratch
# edit Program.cs, then:
dotnet run
```

You need the .NET SDK installed. Check with `dotnet --version`.

## Official entry points

- .NET docs: https://learn.microsoft.com/dotnet/
- C# language reference: https://learn.microsoft.com/dotnet/csharp/
- C# fundamentals: https://learn.microsoft.com/dotnet/csharp/fundamentals/
