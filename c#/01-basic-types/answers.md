# Basic types: answers

## Conceptual

1. They are the same type. `int` is a C# keyword alias for `System.Int32`. You can
   use both names in the same program and mix them freely, for example
   `int a = 1; System.Int32 b = a;`.

2. `int` is `0`, `bool` is `false`, `char` is `'\0'` (the null character),
   `double` is `0.0`, and `string` (a reference type) is `null`.

3. `double` is binary floating point. `0.1` has no exact representation in base 2,
   like `1/3` has no exact decimal form, so small rounding errors appear and
   accumulate. For a bank balance use `decimal`, which is base-10 and stores decimal
   fractions exactly. The trade-off is that `decimal` has a smaller range and is
   slower, so it is not a good fit for heavy scientific math.

4. `var` asks the compiler to infer the static type from the initializer. The
   variable is still statically and strongly typed, not dynamic. `var x = 5;` makes
   `x` an `int` for its whole lifetime. It does not compile without an initializer
   (`var x;`), or with a `null` initializer and no target type, because there is
   nothing to infer from.

5. `int` always holds an integer value. `int?` can hold an integer or `null`. `int?`
   is shorthand for `System.Nullable<int>`. Use `.HasValue`/`.Value` or `??` to work
   with it safely.

6. In a release build the default is unchecked, so `int.MaxValue + 1` wraps around
   to `int.MinValue` with no error. Wrap the expression in `checked(...)`, or enable
   checked arithmetic for the project, to get an `OverflowException`.

7. `42` is `int`, `42L` is `long`, `3.14` is `double`, `3.14f` is `float`, `3.14m`
   is `decimal`, `'a'` is `char`, `"a"` is `string`.

8. No. `bool` is its own type with values `true` and `false`, not `1` and `0`.
   `if (1)` does not compile; the condition must be a `bool` expression such as
   `if (x == 1)`.

## Coding

9. Example:

```csharp
int i = 1;
long l = 2L;
double d = 3.14;
decimal m = 3.14m;
bool b = true;
char c = 'x';
string s = "hello";

foreach (object v in new object[] { i, l, d, m, b, c, s })
    Console.WriteLine($"{v} : {v.GetType()}");
```

Putting value types into an `object[]` boxes them, but `GetType()` still reports the
real runtime type such as `System.Int32`.

10. Example:

```csharp
double dSum = 0;
decimal mSum = 0m;
for (int n = 0; n < 10; n++)
{
    dSum += 0.1;
    mSum += 0.1m;
}
Console.WriteLine(dSum);  // 0.9999999999999999
Console.WriteLine(mSum);  // 1.0
```

The `double` result is slightly off because `0.1` is not exact in binary. The
`decimal` result is exactly `1.0`.

11. Example:

```csharp
int? value = null;         // try also: int? value = 7;
Console.WriteLine(value.HasValue ? value.Value.ToString() : "no value");
// or, shorter:
Console.WriteLine(value?.ToString() ?? "no value");
```

12. Example:

```csharp
int max = int.MaxValue;
Console.WriteLine(unchecked(max + 1)); // -2147483648, wraps
try
{
    Console.WriteLine(checked(max + 1));
}
catch (OverflowException)
{
    Console.WriteLine("overflow"); // this runs
}
```
