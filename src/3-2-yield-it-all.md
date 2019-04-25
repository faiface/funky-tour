# Yield it all!

You thought vertical code was just for `IO`? You were wrong! Vertical code (the one that spans vertically and can be read top to bottom) is quite universal in Funky. In this part, we'll see how it can be applied to lists and their generation.

## The `yield` function

The basic list constructor, the double colon, is an infix function. That makes it unusable for vertical code. However, the standard library defines a synonym of `::` called `yield`:

```funky
func yield : a -> List a -> List a = (::)
```

It's a literal synonym. But, it's a prefix function. How does that help?

Take this list:

```funky
[1, 2, 3]
```

Rewrite it with the colons:

```funky
1 :: 2 :: 3 :: empty
```

Cool. Now, how would this look if we used `yield` instead?

```funky
yield 1 (yield 2 (yield 3 empty))
```

All we did was replace all `::`s with `yield`s. And here it comes. Can you see it? We can use the semicolon and make it vertical!

```funky
yield 1;
yield 2;
yield 3;
empty
```

Beautiful!

To get to know `yield` a little better, let's use it to rewrite some well known functions. How would the `map` function look like using `yield`? Something like this:

```funky
func my-map : (a -> b) -> List a -> List b =
    \f \list
    for list (
        \x \next
        yield (f x);
        next
    );
    empty
```

The `for` function isn't just for `IO`. It can be used with anything that composes vertically. We take the function `f` and the `list` and we `yield` all of its elements in order, except we transform them with `f`.

We can even make the code shorter with function composition:

```funky
func my-map : (a -> b) -> List a -> List b =
    \f \list
    for list (yield . f);
    empty
```

That's really nice.

## The `when` function

How would `filter` look like with `yield`? In the case of `filter`, we can't `yield` all the elements of the list. We must only `yield` those that satisfy the predicate. That's where the `when` function comes handy:

```
$ funkycmd -types
> when
Bool -> (a -> a) -> a -> a
```

It's kinda similar to `if`, but different. While `if` chooses between two alternatives, `when` conditionally applies its second argument (the **body**) to the third (the **next**). Specifically, `when` works like this:

```funky
when true body next   # => body next
when false body next  # => next
```

For example:

```funky
when (x > 7) (1 +) x
```

This expression evaluates to `x + 1` if `x` is more than 7, otherwise it evaluates to `x`.

So, let's use `when` to implement `my-filter`!

```funky
func my-filter : (a -> Bool) -> List a -> List a =
    \p \list
    for list (
        \x \next
        when (p x) (yield x);
        next
    );
    empty
```

That's it! Here, we can't easily compress the loop body into some stunning composition, but so what. It isn't that bad anyway.

## Formatting lists

Structuring list generation vertically comes very handy when dealing with complex list generating functions. One of the best examples is a function to format lists so that we can print them with one `println`. Such function isn't currently present in the standard library. But we'll make our own!

First, we gotta choose the type. What could it be? Perhaps this one?

```funky
List a -> String
```

Well, this is what it will be in the future, when the [`with` clause](TODO) gets implemented, but so far, we can't do it that way. Why? We're accepting `List a`: a list of any type. To convert it to string, we need to convert each of its elements to string. But, there's no general way of converting an arbitrary type to string - the `string` function is only implemented for a handful of types.

This will do:

```funky
(a -> String) -> List a -> String
```

In addition to the list itself, we'll accept a function that helps us convert the elements to strings. Okay, let's get to the code!

```funky
func list->string : (a -> String) -> List a -> String =
    \str \list
    # TODO
```

The type `String` is an alias for `List Char` - the list of characters. That's what we have to produce. We start with the opening square bracket:

```funky
func list->string : (a -> String) -> List a -> String =
    \str \list
    yield '[';
    # TODO
```

Now we have to loop over the list and yield the string versions of the elements. We also gotta put commas between them. We gotta put a comma before each element except the first one. How are we going to achieve that?

We'll learn two more functions: `enumerate` and `for-pair`. The `enumerate` function takes a list and adds an index to each element. Its type is `List a -> List (Pair Int a)`. For example: `enumerate ["A", "B", C"]` evaluates to `[pair 0 "A", pair 1 "B", pair 2 "C"]`. You get the idea. Now, `for-pair` lets us easily iterate over a list of pairs. It's just like `for`, but it's body function takes two elements instead of one - `for-pair` unpacks the pairs.

```funky
func list->string : (a -> String) -> List a -> String =
    \str \list
    yield '[';
    for-pair (enumerate list) (
        \i \x \next
        # TODO
    );
    # TODO
```

The `i` variable is the index, the `x` variable is the original element from `list`. If the index is zero, we know we're dealing with the first element and we won't add the comma. Otherwise, we'll add it.

```funky
func list->string : (a -> String) -> List a -> String =
    \str \list
    yield '[';
    for-pair (enumerate list) (
        \i \x \next
        when (i > 0) (yield-all ", ");
        yield-all (str x);
        next
    );
    # TODO
```

What's that `yield-all`? It's just like `yield`, but it yields a whole list of things, instead of just one element. It's a synonym for `++`. In general:

```funky
yield-all list;
next
```

is equivalent to:

```funky
for list yield;
next
```

All that remains now is finishing with the closing square bracket:

```funky
func list->string : (a -> String) -> List a -> String =
    \str \list
    yield '[';
    for-pair (enumerate list) (
        \i \x \next
        when (i > 0) (yield-all ", ");
        yield-all (str x);
        next
    );
    yield ']';
    empty
```

And we're done! It's so nice and readable having this function expressed vertically.

> **Note.** Actually, `list->string` can be done nicer in a functional style using the `join` function.
>
> `"[" ++ join ", " (map str list) ++ "]"`
>
> Functional solutions are often superior to imperative ones. We've done `list->string` imperatively just for the demonstration.

Let's test it out! We'll print all the primes until 100. First, the prime testing function:

```funky
func prime? : Int -> Bool =
    \n
    all (\x not zero?; n % x) (range 2 (n - 1))
```

It simply tests whether no number between 2 and n-1 divides n. The `%` sign is the modulo operator. Now we'll make an infinite list of primes:

```funky
func primes : List Int =
    filter prime? (iterate (+ 1) 2)
```

> **Details.** The function `iterate` successively applies the provided function to the provided initial value producing an infinite list of all the results. In our case, `iterate (+ 1) 2` produces `[2, 3, 4, 5, 6, ...]`. For example, `iterate (* 2) 1` would produce `[1, 2, 4, 8, 16, 32, ...]`.

Okay, now, let's print them up to 100:

```funky
func main : IO =
    let (take-while (< 100) primes) \primes<100
    println (list->string string primes<100);
    quit
```

> **Details.** The `take-while` function is different from `filter`. While `filter` takes _all_ the elements that satisfy a predicate, `take-while` takes elements from the list, but _stops_ taking them the moment it encounters the first element to not satisfy the predicate. That's important here. Had we used `filter`, the program wouldn't halt. It would continue searching the infinite list of primes for another element less than 100, but it would never find one. Using `take-while` makes the program halt, because we never search beyond the first wrong one.

Let's run it!

```funky
$ funkycmd primes.fn
[2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97]
```

Works like charm!

In the next part, we'll learn how to express state flow. Things will get really imperative... in a purely functional way.
