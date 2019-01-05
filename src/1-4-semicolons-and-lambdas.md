# Semicolons and lambdas

You've probably noticed the semicolon we used in the previous part and wondered what that's for. Its meaning is actually very simple. It's used to **separate the last argument** to a function _without_ the need to put it in parentheses. Let's see. Say we have an expression like this:

```funky
string (sqrt 4.0 + sqrt 9.0)
```

We can omit the parentheses and use the semicolon instead:

```funky
string; sqrt 4.0 + sqrt 9.0
```

> **Note.** If you're familiar with Haskell, you'll notice that the semicolon is just like Haskell's `$` operator. That's correct. Funky uses the semicolon instead of the dollar to avoid clutter, because it's used very frequently.

The semicolon becomes very useful when structuring code. We could write this:

```funky
func main : IO =
    println "A" (println "B" (println "C" quit))
```

That's a completely valid code that prints three lines:

```
$ funkycmd abc.fn
A
B
C
```

> **Explanation.** If you're having trouble understand the code above, here's an explanation. The `IO` is a data structure that describes what the program should do. The simplest `IO` value is `quit`. It simply instructs the program to quit.
>
> The type of the `println` function is `String -> IO -> IO`. That means that `println` takes two arguments, a `String` and an `IO`, and evaluates to an `IO`. The resulting `IO` describes a program that first prints the provided string, then does whatever the second argument says.
>
> Let's look at the inner-most (last) usage of `println` in the above code: `println "C" quit`. This expression of type `IO` describes a program that prints the string `"C"` and then quits. Since it is an `IO`, we can pass it as the second argument to another `println` and we get `println "B" (println "C" quit)`. And using the same technique once again, we end up with the code above.

But, we have a lot of parentheses there! Let's get rid of them with those semicolons:

```funky
func main : IO =
    println "A"; println "B"; println "C"; quit
```

> **Note.** We technically don't need the last semicolon as `quit` is just a single word, but we'll leave it there for _style_.

And finally, we can split it into mutliple lines:

```funky
func main : IO =
    println "A";
    println "B";
    println "C";
    quit
```

Now that's cool! It's got lines, it's got semicolons, looks just like an imperative code! Of course, I'm joking a bit here, but it's not that bad, is it?

## The `if` function

**Booleans** have type `Bool`, two values - `true` and `false` - and a handy `if` function for making a choice. What's its type? The beloved types sandbox comes to rescue:

```
$ funkycmd -types
> if
Bool -> a -> a -> a
```

The little `a` is a type variable. You can replace it with any type whatsoever and `if` will conform to that type. Of course, you must replace all occurences of `a` consistently. So `if` can be used as `Bool -> Int -> Int -> Int`, or as `Bool -> IO -> IO -> IO`, but **not** as `Bool -> Int -> IO -> Float`.

The `if` function checks whether the condition (the first argument) is `true` or `false`. If it's `true`, the second argument is returned. Otherwise, the third argument is returned. For example:

```
> if true 1 3            # evaluates to 1
Int
> if (4 < 1) "YES" "NO"  # evaluates to "NO"
List Char
```

> **Note.** Comments start with the `#` symbol. Also notice that `String` and `List Char` are synonyms.

Combining the `if` function with the semicolon, we create an _if, else if, else_ chain:

```funky
if (guess > correct) "Less.";
if (guess < correct) "More.";
"Correct!"
```

Isn't this beautiful? A single function suitable both for a one-line choices and for a series of mutually exclusive branches.

## Lambdas

Funky has a very concise syntax for anonymous functions. That's crucial because just like semicolons, anonymous functions are used all over the place. Here's how you write them:

```funky
\variable result
```

That's it. A backslash, the name the variable, and an expression to evaluate to.

> **Important.** All identifiers in Funky are allowed contain almost arbitrary Unicode characters. That's because tokens are generally separated by whitespace. The only exceptions are the symbols `(`, `)`, `[`, `]`, `{`, `}`, `,`, `;`, `\`, and `#` that are always parsed as separate tokens.
>
> Therefore, **idiomatic naming** in Funky is very similar to the one in LISP. Dashes `are-used` instead `of_underscores`, functions returning `Bool` (predicates) usually end with the `?` symbol, and partial functions (can crash) end with the `!` symbol.

Functions of multiple arguments are just functions that take the first argument and return a function taking the rest of the arguments (currying):

```funky
\x \y x + y
```

Funky uses **lazy evaluation**. That means that arguments to functions are only evaluated when their value is needed. For example, the factorial of 20 is never calculated here:

```funky
(\n 3) (factorial 20)  # evaluates to 3
```

Also, the factorial of 10 is only computed once, not twice here:

```funky
(\n n + n) (factorial 10)
```

## The `let` function

