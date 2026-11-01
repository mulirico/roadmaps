# Basic types: exercises

Try these without looking at `answers.md`. For coding tasks, write and run the code
in a scratch console project.

## Conceptual

1. What is the difference between `int` and `System.Int32`? Can you use both in the
   same program?
2. List the default values for `int`, `bool`, `char`, `double`, and a reference type
   like `string`.
3. Why can a `double` not represent `0.1` exactly, and what type would you use for a
   bank balance? Explain the trade-off.
4. What does `var` do? Is a variable declared with `var` dynamically typed? Give a
   case where `var` does not compile.
5. What is the difference between `int` and `int?`? What is `int?` shorthand for?
6. What happens when `int.MaxValue + 1` is evaluated in a normal release build, and
   how do you make it throw instead?
7. What is the type of each literal: `42`, `42L`, `3.14`, `3.14f`, `3.14m`, `'a'`,
   `"a"`?
8. Is `bool` an integer in C#? Can you write `if (1)`? Explain.

## Coding

9. Write a snippet that declares one variable of each: `int`, `long`, `double`,
   `decimal`, `bool`, `char`, `string`. Print each value and its runtime type using
   `.GetType()`.
10. Show the difference between `decimal` and `double` by summing `0.1` ten times
    with each type and printing both results. Explain what you see.
11. Write code that reads a nullable `int?` and prints the value, or prints
    `"no value"` when it is null, using the `??` operator.
12. Demonstrate `checked` and `unchecked` with `int.MaxValue + 1`. Show one value
    that wraps and one that throws.
