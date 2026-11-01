# Types, value vs reference: exercises

Try these without looking at `answers.md`.

## Conceptual

1. What is the core difference between a value type and a reference type in terms of
   what gets copied on assignment?
2. The claim "value types live on the stack, reference types on the heap" is a
   simplification. Give one case where a value type does not live on the stack.
3. When are two `record` instances considered equal? How does that differ from two
   plain `class` instances?
4. What is boxing? Write a line that boxes and a line that unboxes. Why is boxing a
   performance concern, and how do generics help?
5. For a plain class with no overrides, what do `==`, `Equals`, and
   `ReferenceEquals` compare? What changes for `string`?
6. `string` is a reference type but is often described as behaving like a value.
   Why? What does `s.Replace("a", "b")` return and what happens to the original `s`?
7. When would you choose a `struct` over a `class`? Name two guidelines.
8. Explain the difference between `ref` and `out`. Which one must be initialized by
   the caller before the call, and which must be assigned by the method?

## Coding

9. Write code that shows reference copy semantics: create a class with a mutable
   field, assign one instance to a second variable, change the field through the
   second variable, and show the first variable sees the change.
10. Write code that shows value copy semantics with a `struct`: do the same as above
    and show the first variable does not see the change.
11. Define a `record Person(string Name, int Age)`. Create two instances with the
    same values and show they are equal with `==`. Then use a `with` expression to
    make a copy with a different age.
12. Write a method `bool TryHalve(int n, out int result)` that sets `result` to
    `n / 2` and returns `true` only when `n` is even. Call it and print the outcome
    for an even and an odd input.
