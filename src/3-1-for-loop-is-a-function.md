# For loop is a function

_Lists_. They are the butter and bread of functional programming and possibly my favorite data structure ever.

This is how they are defined in the standard library:

```funky
union List a = empty | a :: List a
```

It's a trivial linked list, which means you can't access elements randomly, you always have to traverse them in order.

As you can see, lists can either be empty, or have a first element and a rest of the list. Using the `::` constructor repeatedly allows for creating longer and longer lists:

```funky
empty
1 :: empty
1 :: 2 :: empty
1 :: 2 :: 3 :: 4 :: 5 :: 6 :: 7 :: empty
```

And using recursion, one can even create infinite ones:

```funky
func one-two-three : List Int = 1 :: 2 :: 3 :: one-two-three
```

This doesn't enter an infinite loop because Funky is a [lazily evaluated language](https://en.wikipedia.org/wiki/Lazy_evaluation).

Of course, typing colons all the time would be tiresome, and so Funky provides a nice syntax suggar with the usual square brackets.

```funky
["a", "b", "c", "d"]
```

Expands to:

```funky
"a" :: "b" :: "c" :: "d" :: empty
```

[Well, actually](https://www.urbandictionary.com/define.php?term=well%20actually)... since the `String` type is defined as `List Char`, the string literals must also get expanded to colons, right? Yep, so the full expansion actually is this monstrosity:

```funky
('a' :: empty) :: ('b' :: empty) :: ('c' :: empty) :: ('d' :: empty) :: empty
```

After seeing this, I'm sure you're way more grateful for all the string and list syntax suggar than you've ever been before.

## All the _"cool functional stuff"_

With lists comes the legacy of tools, perfected and sharpened for long generations. Let me (re)introduce you to `map` and `filter`.

The function `map` simply applies a function to all of the elements of the list:

```funky
map sqrt [1.0, 4.0, 9.0, 16.0]  # => [1.0, 2.0, 3.0, 4.0]
map (* 3) [5, 1, 3, 2]          # => [15, 3, 9, 6]
```

The function `filter` takes a predicate and filters out all the elements that don't satisfy it:

```funky
filter (< 5) [9, 3, 4, 7, 5, 10, 1]  # => [3, 4, 1]
filter even? [1, 2, 3, 4, 5, 6, 7]   # => [2, 4, 6]
```

Then there is a nice `zip` function which "zips" two lists with some binary function (it's like `zipWith` from Haskell; the `zip` function from Haskell can be done with `zip pair` in Funky). It takes the first elements from both lists, applies the function to them, and goes on, like this:

```funky
zip (*) [1, 2, 3] [2, 3, 4]           # => [2, 6, 12]
zip (++) ["he", "wo"] ["llo", "rld"]  # => ["hello", "world"]
```

The `zip` function enables this ultra cool hipster fibonacci one liner:

```funky
func fibonacci : List Int =
    0 :: 1 :: zip (+) fibonacci (rest! fibonacci)
```

> **Details.** The `rest!` function returns the list except its first element and crashes if the list is empty, therefore the `!` sign. For example, `rest! [1, 2, 3]` evaluates to `[2, 3]`. Similarly, there is a `first!` function that evaluates to the first element of the list.
>
> These functions also have their non-crashing versions without the `!` signs that return `Maybe`s (optional values, we'll learn more about them). For example, `first ["a", "b", "c"]` evaluates to `some "a"`, while `first []` evaluates to `none`.

There are  more list functions, like `fold<` and `fold>` (those are same as `foldr` and `foldl'` in Haskell), but we'll not give them too much attention, you can try them out on your own. For example, here's how you'd sum a list of numbers with `fold>`:

```funky
fold> (+) 0 (range 1 15)  # => 120
```

Other list functions include `reverse`, `take`, `drop`, `any`, `all`, `iterate`, and so on. The usual ones.

## How to print a list? Meet the for loop

Now, all that stuff is cool, but if we can't print a list, it's all useless, right? Well, this leads us to a very interesting function from the standard library called `for`. Yes, the name is intentionally chosen to match the name of the old imperative concept - the for loop. Of course, loops in imperative programming rely on mutation of the loop variables. We can't do that in functional programming, but we'll come pretty close.

So, what's the type of this mighty `for` function?

```
$ funkycmd -types
> for
List a -> (a -> c -> c) -> c -> c
```

So, it takes three arguments. Here's what they are and how you can think about them:

- **List.** This is the list of things we want to loop over.
- **Body.** You can think of this as something we want to "do" for each element.
- **Next.** This will get "done" after getting over with all the elements.

Obviously, the descriptions above are rather innacurate. We can't _do_ anything, we can only make values and data structures.

The best way to explain what `for` does is by an example. So, **let's print the list**:

```funky
func main : IO =
    for ["A", "B", "C"]
        println;
    quit
```

Oh man, that looks imperative! Does it even work?

```
$ funkycmd for.fn
A
B
C
```

It does!

We've passed `["A", "B", "C"]` as the list to `for`. Then we passed `println` as the body argument. The `println` function perfectly fits the required type of the argument: `a -> c -> c`, specialized to `String -> IO -> IO` in our case. Because `println` takes and returns `IO`, the last argument to `for` must be an `IO`. So we passed `quit`.

As we already know, `println` constructs the `IO` data structure and the same goes for `quit`. So `for` is in fact just applying functions in some way. What could that way be?

So, first of all, if we pass an empty list, we just get the **next** back.

```funky
for [] body next  # => next
```

If we pass a single element list, we get the **body** applied to the element and the next:

```funky
for [a] body next  # => body a next
```

What about a two element list? Well, that looks like this:

```funky
for [a, b] body next  # => body a (body b next)
```

Three elements?

```funky
for [a, b, c] body next  # => body a (body b (body c next))
```

In other words:

```funky
for (x :: xs) body next
```

is equivalent to:

```funky
body x (for xs body next)
```

which can be easily rewritten to:

```funky
body x;
for xs body;
next
```

Knowing this, it's easy to see that our list printing program:

```funky
func main : IO =
    for ["A", "B", "C"]
        println;
    quit
```

basically expands to this (during runtime, of course):

```funky
func main : IO =
    println "A";
    println "B";
    println "C";
    quit
```

It's all very clear now!

The **next** argument doesn't have to be just a `quit`. It can be more complex:

```funky
func main : IO =
    for ["A", "B", "C"]
        println;
    println "Done!";
    quit
```

or even another `for` loop:

```funky
func main : IO =
    for ["A", "B", "C"]
        println;
    println "First half!";
    for ["D", "E", "F"]
        println;
    println "Second half!";
    quit
```

With the semicolon, we can get much of the convenient code structuring of imperative programming. **Without monads!**

### How to print lists of numbers?

If we change the list to a list of numbers, the scheme no longer works:

```funky
func main : IO =
    for (range 1 5)
        println;
    quit
```

> **Details.** The `range` function takes the lower and the upper bound and evaluates to a list of all integers in between, including the bounds.

Trying to run this gives us an error:

```
$ funkycmd for.fn
for.fn:3:9: type-checking error
```

That's quite obvious. We have a list of `Int`s, but `println` takes a `String`. This can be fixed by composing the `println` function with the `string` function, which converts `Int`s (and is overloaded for other types) to `String`s:

```funky
func main : IO =
    for (range 1 5)
        (println . string);
    quit
```

This time, it works!

```
$ funkycmd for.go
1
2
3
4
5
```

### How to put more things in a body?

So far, we've only seen a simple function, or a composition of two functions as the body argument to `for`. What if we want to have a more elaborate loop body? Say a two `println`s for the start.

You might be tempted to try this:

```funky
func main : IO =
    for (range 1 5)
        (print "#"; println);
    quit
```

But that just doesn't work. We can't pass `println` as the second argument to `print`, that's just nonsense, it expects an `IO`, not `String -> IO -> IO`.

**Here's what we need to do.** Let's start with the simple for loop on strings.

```funky
func main : IO =
    for ["A", "B", "C"]
        println;
    quit
```

Instead of just passing the function `println` itself, let's make it a lambda:

```funky
func main : IO =
    for ["A", "B", "C"] (
        \s \next
        println s;
        next
    );
    quit
```

The body of the `for` loop takes two arguments: the _element_ of type `String`, and the _continuation_ of type `IO`. In our case, the continuation is `quit` for the last element of the list, and for the rest of the elements it works just as described above. We pass both of these to the `println` function, recreating the original, short for loop. This doesn't at all change the behavior of the program.

However, as you can surely see, it's very straightforward to add more "statements" to the body:

```funky
func main : IO =
    for ["A", "B", "C"] (
        \s \next
        print "- ";
        println s;
        next
    );
    quit
```

Let's run it!

```
$ funkycmd body.fn
- A
- B
- C
```

We can even add scans to the loop!

```funky
func main : IO =
    print "What's the number of questions? ";
    scanln \n-str
    let (int n-str) \n
    for (range 1 n) (
        \i \next
        print ("Your question #" ++ string i ++ ": ");
        scanln \question
        println "The answer: IDK.";
        next
    );
    quit
```

Let's run this funny program:

```
$ funkycmd questions.fn
What's the number of questions? 3           
Your question #1: What's your name?
The answer: IDK.
Your question #2: How are you doing IDK?
The answer: IDK.
Your question #3: Do you know anything?
The answer: IDK.
```

And you can even nest loops! Here's a program that prints out the small multiplications table... in a non-tabular form, but that's okay.

```funky
func main : IO =
    for (range 1 10) (
        \x \next
        for (range 1 10) (
            \y \next
            print (string x);
            print " x ";
            print (string y);
            print " = ";
            println (string (x * y));
            next
        );
        next
    );
    quit
```

> **Note.** Those multiple prints aren't necessary, it could've all been done with one long string concatenation. Splitting it into multiple lines just makes it clearer. This problem will go away once there is a better way to format strings.

And run it!

```
$ funkycmd multiplications.fn
...
5 x 7 = 35
5 x 8 = 40
5 x 9 = 45
5 x 10 = 50
6 x 1 = 6
6 x 2 = 12
6 x 3 = 18
6 x 4 = 24
6 x 5 = 30
6 x 6 = 36
6 x 7 = 42
6 x 8 = 48
6 x 9 = 54
6 x 10 = 60
7 x 1 = 7
7 x 2 = 14
7 x 3 = 21
7 x 4 = 28
7 x 5 = 35
...
```

It works! It's the whole output, because that's too large, but you get the idea.

That's it for this part, in the next part, we'll learn more list tricks!
