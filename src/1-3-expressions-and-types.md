# Expressions and types

Funky is a strongly typed language. To explore types, Funky provides a _types sandbox_. It's kind of like a [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop), but it doesn't evaluate anything - just tells you the types.

Here's how we fire it up:

```
$ funkycmd -types
>
```

It opens up a little prompt where we can type expressions. Let's try it!

```
$ funkycmd -types
> 5
Int
> 3.14
Float
> pi
Float
> 'x'
Char
```

> **Details.** Funky has three built-in types: `Int`, `Float`, and `Char`. `Int`s are arbitrary-precision integers. `Float`s are IEEE-754 64-bit floating-point numbers. `Char`s are [Unicode](https://en.wikipedia.org/wiki/Unicode) characters. All other types are either from the standard library or defined by the programmer.

How about using some operators and functions?

```
> 13 + 48
Int
> sqrt 4
sandbox:1:6: type-checking error
```

Ugh, what? A _type-checking error_? Yeah, that's right.

```
> sqrt
Float -> Float
> 4
Int
```

As we can see, the type of the `sqrt` function is `Float -> Float` (a function that takes a `Float` and returns a `Float`), while the value we supplied is an `Int`. This doesn't match. Instead, we need to supply a `Float` value:

```
> sqrt 4.0
Float
```

Now this works.

> **Details.** There are many built-in functions for each of these built-in types. Arithmetic operators (`+`, `-`, `*`, `/`, `^`, `%`), comparison operators (`==`, `!=`, `<`, `>`, `<=`, `>=`), math functions (`sin`, `cos`, `sqrt`, `atan2`, etc.). One notable thing is that you can't add characters - `'a' + 'b'` is not valid - but you can add an integer to a character: `'a' + 2` results in `'c'`.

> **Note**. The compiler error messages will get better and more descriptive.

## Printing various values

We've already printed `"Hello, world!"`, how about printing other stuff? Let's try:

```funky
func main : IO =
    println 13;
    quit
```

Try running it:

```
$ funkycmd hello.fn
hello.fn:2:13: type-checking error
```

A type-checking error again? Let's see where we went wrong. Start up the types sandbox:

```
$ funkycmd -types
>
```

And let's examine the type of the `println` function:

```
> println
String -> IO -> IO
```

Oh, so the `println` function only takes strings. Could've guessed that!

> **Note.** Althought it might not seem like it at first, `println` is a completely pure function. It takes two arguments: a `String` and an `IO`. The `IO` is a continuation - it says what should be done after the string gets printed. In our case, the continuation is `quit` (which has the type `IO` itself). This _instructs_ the program to quit after printing the string. The precise workings of this will be explained soon.

So, how do we convert an `Int` or a `Float` to a `String` so that we can print it?

The `string` function is for that purpose. Let's see what's the type of this `string` function:

```
$ funkycmd -types
> string
Int -> String
Float -> String
Bool -> String
Char -> String
String -> String
```

Wait, what? The `string` function has multiple types? That's exactly correct!

Unlike most functional languages, Funky supports **overloading**. That means that you can define many functions with the same name, provided they are _distinguishible_ by their types.

Back to the printing. This now works:

```funky
func main : IO =
    println (string 13);
    quit
```

Runs as expected:

```
$ funkycmd hello.fn
13
```

> **Exercise.** Play with various functions. Try operators like `*`, `^`, `%`, math functions like `sqrt`, `sin`, `log`, booleans `true` and `false`, string functions `++`, `*`, and so on. The types sandbox tells you what exists and what types it has.
