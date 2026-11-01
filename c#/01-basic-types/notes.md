# Basic types in C#

## What this covers

The built-in types every C# developer uses daily, their sizes and ranges, how
literals work, type inference with `var`, default values, nullable value types, and
the choices that come up in interviews (for example `decimal` vs `double`).

## Built-in value types

C# keywords are aliases for types in the `System` namespace. `int` is
`System.Int32`, `bool` is `System.Boolean`, and so on. They are the same type, the
keyword is just a shorter name.

Integral types:

| Keyword  | System type     | Size    | Signed |
|----------|-----------------|---------|--------|
| `byte`   | `System.Byte`   | 8 bit   | no     |
| `sbyte`  | `System.SByte`  | 8 bit   | yes    |
| `short`  | `System.Int16`  | 16 bit  | yes    |
| `ushort` | `System.UInt16` | 16 bit  | no     |
| `int`    | `System.Int32`  | 32 bit  | yes    |
| `uint`   | `System.UInt32` | 32 bit  | no     |
| `long`   | `System.Int64`  | 64 bit  | yes    |
| `ulong`  | `System.UInt64` | 64 bit  | no     |

Floating-point and decimal:

| Keyword   | System type      | Notes                                         |
|-----------|------------------|-----------------------------------------------|
| `float`   | `System.Single`  | 32 bit, binary floating point, suffix `f`     |
| `double`  | `System.Double`  | 64 bit, binary floating point, default for real literals |
| `decimal` | `System.Decimal` | 128 bit, base-10, suffix `m`, for money       |

Other value types: `bool` (`true` or `false`, not an integer), `char`
(`System.Char`, a single UTF-16 code unit).

## string

`string` (`System.String`) is a reference type, not a value type, but it behaves in
a value-like way because it is immutable. Any operation that looks like it changes a
string returns a new string. It is listed here because it is a built-in type with
its own keyword.

## Literals and suffixes

- `42` is an `int`. `42L` is a `long`. `42U` is `uint`.
- `3.14` is a `double`. `3.14f` is a `float`. `3.14m` is a `decimal`.
- `0xFF` is hex, `0b1010` is binary, `1_000_000` uses digit separators.
- `'a'` is a `char`, `"abc"` is a `string`.

## var and type inference

`var` tells the compiler to infer the type from the right-hand side. It is still
statically typed. `var x = 5;` makes `x` an `int`, and it cannot later hold a string.
`var` needs an initializer, so `var x;` does not compile.

## Default values

Every type has a default value, used for unassigned fields and array elements:

- Numeric types default to `0` (or `0.0`).
- `bool` defaults to `false`.
- `char` defaults to `'\0'`.
- Reference types default to `null`.

You can write `default(int)` or `default` in a typed context.

## Nullable value types

Value types cannot hold `null` by default. `int?` (shorthand for
`System.Nullable<int>`) can hold either an `int` or `null`. Use `.HasValue` and
`.Value`, or the null-coalescing operator `??` to supply a fallback.

```csharp
int? age = null;
int shown = age ?? 0; // 0 when null
```

## Overflow: checked and unchecked

By default, integer arithmetic that overflows wraps around silently in a release
build. `checked` forces an `OverflowException`; `unchecked` forces wrap-around even
where the project enables checked arithmetic.

```csharp
int max = int.MaxValue;
int wrap = unchecked(max + 1);     // int.MinValue
int boom = checked(max + 1);       // throws OverflowException
```

## decimal vs double

`double` is binary floating point, so values like `0.1` are not represented exactly
and errors accumulate. `decimal` is base-10 and represents decimal fractions
exactly, at the cost of range and speed. Use `decimal` for money and anything where
exact decimal values matter. Use `double` for scientific and general real-number
math.

## Documentation

- Built-in types: https://learn.microsoft.com/dotnet/csharp/language-reference/builtin-types/built-in-types
- Integral numeric types: https://learn.microsoft.com/dotnet/csharp/language-reference/builtin-types/integral-numeric-types
- Floating-point numeric types: https://learn.microsoft.com/dotnet/csharp/language-reference/builtin-types/floating-point-numeric-types
- bool: https://learn.microsoft.com/dotnet/csharp/language-reference/builtin-types/bool
- char: https://learn.microsoft.com/dotnet/csharp/language-reference/builtin-types/char
- Nullable value types: https://learn.microsoft.com/dotnet/csharp/language-reference/builtin-types/nullable-value-types
- Default values of C# types: https://learn.microsoft.com/dotnet/csharp/language-reference/builtin-types/default-values
- Implicitly typed local variables (var): https://learn.microsoft.com/dotnet/csharp/programming-guide/classes-and-structs/implicitly-typed-local-variables
