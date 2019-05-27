# Functions

So far, we've only seen one function defined: the `main` function. But even from that alone, I'm sure you've already guessed how function definitions look. They're quite simple:

```funky
func name : type = body
```

That's it!

The **name** can be any string of non-whitespace characters excluding keyword symbols like parentheses, a backslash, a semicolon, and a few others.

The **type** specifies the type that the function name will acquire.

The **body** is an arbitrary Funky expression, functions, lambda, etc. can be used. The only limitation is that it must conform to the type.

> **Details.** Let's talk a bit about types.
>
> As we've already learned, there are three built-in types: `Int`, `Float`, and `Char`. There are also a few types from the standard library, such as `Bool`, or `String`.
>
> Then there are types that take other types as arguments - types that are, so-called, generic. The _most important_ of such generic types is the function type. This is also actually built-in, but since it's a type constructor, rather than a full, distinct type, I haven't included it among the built-in types previously.
>
> The function type constructor is called 'the arrow', written as: `->`. It takes two arguments, the input type on the left and the output type on the right. For example, `Int -> Bool` is a function that takes an `Int` and evaluates to a `Bool`.

So, let's define a function! Every functional language tutorial _must_ include a factorial example, so let's get that behind us:

```funky
func factorial : Int -> Int =
    \n
    if (n <= 0) 1;
    n * factorial (n - 1)

func main : IO =
    print "Type a number: ";
    scanln \n-str
    let (int n-str) \n
    println (string; factorial n);
    quit
```

A standard, well known, recursive factorial definition.

> **Note.** In Funky it's a convention to put arguments on the next line after `=` in the function definition. If the function is very short, this convention may be broken and the whole definition may sit on a single line.

> **Details.** The order of definitions doesn't matter. A function doesn't have to be 'defined before use'. All that matters is that it is defined.

Let's try it:

```
$ funkycmd factorial.fn
Type a number: 5
120
```

If it gives `120` for `5` then it surely works. Let's try something bigger:

```
$ funkycmd factorial.fn
Type a number: 99
933262154439441526816992388562667004907159682643816214685929638952175999932299156089414639761565182862536979208272237582511852109168640000000000000000000000
```

Whoa, that's a number! You can try entering even bigger inputs, but don't go too big... you may freeze your system.

**Now, how about functions with multiple arguments?** Say we want to make a function called `divides` that tells us whether one integer divides another or not.

In Funky, as in many other functional languages, there are no functions of more than one argument. Instead, there are functions that return functions. A function of two arguments really is a function that takes the first argument and returns a new function. This new function remembers the first argument and takes the second one.

What would be the type of such a function?

The type `Int -> (Int -> Bool)` would do. It takes the first `Int`, evaluates to a function taking the second `Int` and finally evaluates to `Bool`.

Great, now how would we call such a function?

Using `(divides 9) 3` would obviously work. But, we don't really need the parentheses. If parentheses are not present, they're implicitly from the left, so `divides 9 3` is good too.

Furthermore, the parentheses in the type `Int -> (Int -> Bool)` aren't needed either. Since `->` is used infix, it's automatically parenthesized from the right, so `Int -> Int -> Bool` is equivalent.

> **Details.** There are two kinds of functions in Funky: prefix and infix. Infix functions can be identified easily: they don't contain any letters. All other functions are prefix.
>
> Applications of prefix functions are automatically parenthesized from the left if no explicit parenthesis are present. In all cases, `f x y` is the same as `(f x) y`.
>
> Applications of infix functions are, however, automatically parenthesized from the right. All of them have the same precedence. This is to simplify the programmer's life: when defining own infix functions, you won't be bothered by specifying precedence levels or associativity. So, `3 * 2 + 1` is equivalent to `3 * (2 + 1)`.

After learning about automatic parentheses (also called left/right-associativity), here's the definition of `divides`:

```funky
func divides : Int -> Int -> Bool =
    \m \n
    (n % m) == 0
```

The body is just two lambdas nested inside one another, which perfectly reflects the type.

We could've also called the function `/?`, in which case it would be an infix function. However, the name `divides` is much more descriptive.

**Function types may also contain type variables**. These are distinguished from actual types by being all lower-case. Actual types must contain an upper-case character - type variables must not.

When a function has a type variable in its type, that means that this function is fine with whatever type instead of that variable. For example, there's this (probably controversially named function, usually it's called `id`) function called `self`:

```
$ funkycmd -types
> self
a -> a
```

It doesn't do anything. It just returns whatever it was passed. For example, `self 4` is `4`. Now, when we passed `4` to `self`, its type changed a bit. It specialized. It became `Int -> Int`. See? We get this specialization just by replacing `a` with `Int` in the type. That's how type variables work.

## Overloading

Words in natural language usually don't have just one meaning: "If only there was a _way_ we could continue that _way_, we could've been _way_ ahead of them, but in a _way_, we haven't done so bad." The previous sentence used four different meanings of the word 'way'.

Using the same words for different, or slightly different meanings in different contexts allows for very concise and expressive speech. Imagine that we'd have to invent a brand new word every time we'd like to add a new meaning to an existing word. That would be cumbersome and unnatural.

Programming is no different. When designing Funky, I realized that support for function overloading brings so many benefits that I couldn't ignore it despite the initial difficulty of implementation.

So, let's try it!

```funky
func double : Int -> Int =
    \x x * 2

func double : Float -> Float =
    \x x * 2.0

func main : IO =
    println (string; double 9);
    println (string; double 4.3);
    quit
```

There we go, two versions of `double`: one for `Int`s, one for `Float`s. Let's run it!

```
$ funkycmd double.fn
18
8.6
```

Works like charm!

Now, how about defining two functions like this?

```funky
func zero : Int = 0

func zero : Float = 0.0

func main : IO =
    println (string zero);
    quit
```

This time, both versions of `zero` fit the context because `string` works for both `Int`s and `Float`s. What does the type checker say?

```
$ funkycmd zeros.fn
zeros.fn:6:21: ambiguous, multiple admissible types:
  Int
  Float
```

That's right, the type checker complains. We can fix this error with a type annotation. You can add an explicit type to any expression using the `:` symbol:

```funky
func zero : Int = 0

func zero : Float = 0.0

func main : IO =
    println (string (zero : Int));
    quit
```

All is fine this time:

```
$ funkycmd zeros.fn
0
```

Well, okay, those two `zero` functions are distinguishable by their type. But what if we put the same type?

```funky
func lucky-number : Int = 7
func lucky-number : Int = 3

func main : IO =
    println (string lucky-number);
    quit
```

Which one gets printed? `7` or `3`? Make your guesses, ladies, and gentlemen...

Here's what happens:

```
$ funkycmd lucky-number.fn
lucky-number.fn:2:27: function lucky-number with colliding type exists: test.fn:1:27
```

Oh, no! It didn't even let us define the second function, because its type collides with the first one.

**Funky doesn't let you overload a function if its type collides with an existing version.**

> **Details.** When exactly do two types collide?
> 
> Do these two functions collide?
> 
> ```funky
> func weirdo : a -> a     = self
> func weirdo : Int -> Int = (* 2)
> ```
> 
> Indeed, they do! You might argue that the second one is more specific than the first one, so if both fit the context, the second one should be selected, but this doesn't fly in Funky, because it brings a whole bag of problems.
> 
> Instead, I chose simplicity: **two types collide whenever their type variables can be substituted such that they become the same type**.
> 
> For example: `a -> Int` collides with `Float -> a` (substitutes to `Float -> Int`).
