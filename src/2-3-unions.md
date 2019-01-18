# Unions

The second way of creating _new_ types is unions. While a record is a collection of fields, a union defines multiple alternative forms.

Here's the general form:

```funky
union Name = alternative argument ... | ...
```

The three dots after `argument` mean that a single alternative can contain any number of arguments (fields) and the three dots after `|` mean that a union can have any number of alternatives.

Unlike records, arguments for alternatives have no names, only types.

Here's a simple union:

```funky
union Bool = true | false
```

This is the actual definition of the type `Bool` from the standard library. The alternatives have no arguments in this case.

Here's another example:

```funky
record Point = x : Float, y : Float

union Shape =
    line Point Point   |
    circle Point Float |
```

> **Note.** Trailing `|` is allowed.

The `Shape` type has two alternatives: the `line` alternatives, which is composed of two points, and the `circle` alternatives, which has a point (the center) and a float (the radius).

When we define a union, Funky **generates a constructor function for each alternative**. The function simply takes all the arguments in order. Let's check them out!

```
$ funkycmd -types shapes.fn
> line
Point -> Point -> Shape
> circle
Point -> Float -> Shape
> line (Point 1.0 8.0) (Point 12.5 -4.9)
Shape
> circle (Point 0.0 0.0) 7.0
Shape
```

They definitely work!

**To examine a `Shape` value and make a decision based on whether it's a line or a circle, Funky provides the `switch`/`case` construct.** It's best shown by an example. Here's a function that calculates the length of a line or a circumference of a circle:

```funky
func length : Shape -> Float =
    \shape
    switch shape
    case line \from \to
        hypot (x to - x from) (y to - y from)
    case circle \center \radius
        2.0 * pi * radius
```

> **Note.** The `hypot` function stands for 'hypotenuse' and is used for calculating the length of the long side of a right triangle using the Pythagorean theorem: `hypot x y = sqrt ((x ^ 2.0) + (y ^ 2.0))`.

As you can see, the `switch`/`case` construct begins with the `switch` keyword followed by the value we're switching on. After that, we see a series of `case` branches. Each branch starts with the `case` keyword followed by the name of the alternative. That is followed by a function that takes the arguments to the alternative and evaluates to the overall result.

Of course, all `case` branches must evaluate to the same type: `Float`, in our case.

> **Note.** Currently, all alternatives must be present in `switch`/`case` and they must be listed in the same order as they are in the definition of the union. This will be fixed.

**The names of the alternatives can also be infix** if they don't contain any letters. When they are infix, they can be placed between their arguments in the definition.

For example, here's a nice, recursive union for building up arithmetic expressions:

```funky
union Expr =
    num Int     |
    Expr + Expr |
    Expr * Expr |
```

It's either just a number, like `num 12`, or it's an addition, or a multiplication: `num 12 + num 7`, or `(num 1 + num 9) * ((num 4 + num 3) + (num 2 * num 2))`.

> **Note.** The alternatives can be called `+` and `*` without a problem thanks to overloading.

To evaluate an expression like that, we make use of `switch`/`case` again:

```funky
func eval : Expr -> Int =
    \expr
    switch expr
    case num
        self
    case (+) \left \right
        eval left + eval right
    case (*) \left \right
        eval left * eval right
```

Have you noticed that no lambda comes after `case num`? That's because the `case` bodies need not explicitly contain lambdas, all they need is a function. In this case, `self` (the identity function) is the right choice because it just evaluates to the number under `num`.

Let's see if `eval` works right!

```funky
union Expr =
    num Int     |
    Expr + Expr |
    Expr * Expr |

func eval : Expr -> Int =
    \expr
    switch expr
    case num
        self
    case (+) \left \right
        eval left + eval right
    case (*) \left \right
        eval left * eval right

func main : IO =
    let ((num 1 + num 9) * ((num 4 + num 3) + (num 2 * num 2))) \expr
    println (string; eval expr);
    quit
```

And run it:

```
$ funkycmd expr.fn
110
```

Works great!

## Type variables in types

So far we've only defined types that were concrete, with no type variables anywhere. But it's also possible (and very useful) to define _generic types_: types parameterized by one or more type variables.

It's really simple.

All we need to do is to list the type variables after the name of the type in the definition:

```funky
record Pair a b =
    first  : a,
    second : b,
```

That's an actual definition of the `Pair` type from the standard library.

Or here's another example:

```funky
union Maybe a = none | some a
```

That is, likewise, the definition of the `Maybe` type from the standard library.

This way, `Pair` and `Maybe` don't define a single type each. Instead, each defines a family of types: `Pair Int Float`, `Pair String Bool`, `Maybe (String -> String)`, `Maybe (Pair Int Int)` all belong to those families.

> **Note.** 'Type family' is not a concept in Funky, it's just a phrase we use to talk about types. `Pair` without its type variables filled in is always an error.
