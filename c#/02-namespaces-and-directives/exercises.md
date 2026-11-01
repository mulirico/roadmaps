# Namespaces and directives: exercises

Try these without looking at `answers.md`.

## Conceptual

1. What problem do namespaces solve? Give an example of two types with the same
   name living in different namespaces.
2. What is the difference between a block-scoped and a file-scoped namespace? What
   is a limitation of the file-scoped form?
3. Explain the difference between a `using` directive and a `using` statement. Do
   they have anything in common besides the keyword?
4. What does `global using` do and where would you put such directives?
5. What are implicit usings, and why can a fresh console app call
   `Console.WriteLine` without writing `using System;`?
6. What does `using static System.Math;` let you write? Give a before and after.
7. You import two namespaces and both define a type named `Timer`. The code does
   not compile. Name two ways to fix it.
8. What is the fully qualified name of `StringBuilder`, and when would you need to
   write it out in full?

## Coding

9. Write a small file that uses a file-scoped namespace `Shop.Orders` and declares a
   class `Order`. Then show the same code with a block-scoped namespace.
10. Write a `using` alias that lets you refer to `System.Collections.Generic.List<T>`
    as `GList<T>`, and use it to create a `GList<int>`.
11. Rewrite this using `using static`:

```csharp
double h = System.Math.Sqrt(System.Math.Pow(3, 2) + System.Math.Pow(4, 2));
```

12. Write a `using` declaration that opens a `StreamWriter`, writes one line, and
    relies on automatic disposal at the end of the scope. Explain when `Dispose` is
    called.
