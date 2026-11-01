# Types: value vs reference

## What this covers

The C# type system from the angle interviewers care about: the split between value
types and reference types, how they are copied and compared, `class` vs `struct` vs
`record`, boxing, equality, string immutability, and parameter passing with `ref`,
`out`, and `in`.

## Value types vs reference types

- Value types hold their data directly. Assigning one to another copies the value.
  Examples: all numeric types, `bool`, `char`, `enum`, and any `struct`.
- Reference types hold a reference to data stored elsewhere. Assigning one copies
  the reference, so both names point at the same object. Examples: `class`,
  `interface`, `delegate`, arrays, and `string`.

The common shorthand is "value types live on the stack, reference types on the
heap". It is a useful first picture but not a rule. A value type that is a field of
a class lives on the heap inside that object, and captured locals can move to the
heap too. What is guaranteed is the copy semantics above, not a storage location.

## class vs struct vs record

- `class`: reference type. Use for most objects, especially larger ones or ones with
  identity and mutable state.
- `struct`: value type. Use for small, immutable, value-like data (a point, a money
  amount). Copied on assignment, so keep them small.
- `record` (C# 9): a reference type with value-based equality generated for you, plus
  `with` expressions for non-destructive copies and a readable `ToString`. Good for
  immutable data models and DTOs.
- `record struct` (C# 10): a value type with the same record conveniences.

The headline difference between a plain `class` and a `record` is equality: two
records are equal when their contents match, while two class instances are equal only
when they are the same reference, unless the class overrides equality.

## Boxing and unboxing

Boxing wraps a value type in an object on the heap so it can be treated as a
reference type (for example stored in an `object` or a non-generic collection).
Unboxing extracts it back. Both cost an allocation and a copy, so boxing in hot
paths is a performance concern. Generics (for example `List<int>`) avoid boxing.

```csharp
int n = 5;
object boxed = n;        // boxing
int back = (int)boxed;   // unboxing
```

## Equality

- `==` on reference types compares references by default (are these the same
  object?), unless the type overloads it. `string` overloads `==` to compare
  contents.
- `==` on value types compares values.
- `object.Equals` can be overridden to give value-based equality for a class.
- `object.ReferenceEquals(a, b)` always checks reference identity and ignores any
  overloads.

So for a plain class, `a == b` and `a.Equals(b)` are reference checks unless
overridden; for a record they are value checks by default.

## string immutability

`string` is a reference type but immutable. Once created, its contents never change.
Methods like `Replace`, `ToUpper`, and concatenation return new strings. Building a
string in a loop with `+` creates many throwaway strings; use `StringBuilder`
instead. Immutability is why strings can be compared by value and shared safely.

## Passing parameters: value, ref, out, in

By default arguments are passed by value. For a reference type this copies the
reference, so the method can mutate the shared object but cannot make the caller's
variable point somewhere else.

- `ref`: pass a variable by reference; the method can read and reassign it, and it
  must be initialized before the call.
- `out`: pass by reference for output; the method must assign it before returning,
  and it need not be initialized before the call.
- `in`: pass by reference as read-only, used to avoid copying a large `struct`
  without allowing changes.

```csharp
void Swap(ref int a, ref int b) { (a, b) = (b, a); }
bool TryParse(string s, out int value) { /* assigns value */ }
```

## enum

An `enum` is a value type whose values are named constants backed by an integral
type (`int` by default). Use it for a fixed set of options. You can cast between the
enum and its underlying type, and `[Flags]` supports bitwise combinations.

## Documentation

- Value types: https://learn.microsoft.com/dotnet/csharp/language-reference/builtin-types/value-types
- Reference types: https://learn.microsoft.com/dotnet/csharp/language-reference/keywords/reference-types
- Classes: https://learn.microsoft.com/dotnet/csharp/fundamentals/types/classes
- Structure types: https://learn.microsoft.com/dotnet/csharp/language-reference/builtin-types/struct
- Records: https://learn.microsoft.com/dotnet/csharp/language-reference/builtin-types/record
- Boxing and unboxing: https://learn.microsoft.com/dotnet/csharp/programming-guide/types/boxing-and-unboxing
- Equality comparisons: https://learn.microsoft.com/dotnet/csharp/programming-guide/statements-expressions-operators/equality-comparisons
- ref, out, in parameter modifiers: https://learn.microsoft.com/dotnet/csharp/language-reference/keywords/method-parameters
- Enumeration types: https://learn.microsoft.com/dotnet/csharp/language-reference/builtin-types/enum
