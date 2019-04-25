# On being exceptional

What a better way to finish a tour than by talking about failing, right? Different languages solve the problem of failure differently. Many use the concept of _[exceptions](https://en.wikipedia.org/wiki/Exception_handling)_: when an exception occurs, the program breaks its control flow and starts back-tracking the calls until it finds an appropriate exception handler. Other languages, such as [Go](https://en.wikipedia.org/wiki/Go_(programming_language)) do it differently, they employ error values that the programmer must manually check.

In this part, we'll talk about how error handling can be done in Funky. Of course, just like pretty much anything else, error handling isn't built-in to the language but is a part of the standard library instead.

**The standard library currently only implements the simplest form of error handling: the `Maybe` type.** More sophisticated error types will be added when the need and specific use-cases arise.

## The `Maybe` type

Most functions always succeed. But some can also fail. To represent failure, we need a new type. The simplest (and so far only) type for this purpose is `Maybe`:

```funky
union Maybe a = none | some a
```

It has two alternatives: **none** and **some**. The former is a failure. The latter is a success.

Do you remember the list manipulation functions `first!` and `rest!`? Well, they crash when the list is empty. Crashing the whole program isn't usually desirable and so there are non-crashing alternatives: `first` and `rest`. What do they return? A `Maybe` of course:

```funky
func first : List a -> Maybe a
func rest  : List a -> Maybe (List a)
```

For example:

```funky
func main : IO =
    let [1, 2, 3, 4] \nums
    println (
        switch first nums
        case none
            "nothing there"
        case some \x
            "the first is " ++ string x
    );
    quit
```

It works:

```
$ funkycmd maybe.fn
the first is 1
```

If we don't wanna use the `switch`, we can use `none?` and `extract!` instead:

```funky
func main : IO =
    let [1, 2, 3, 4] \nums
    println (
        let (first nums) \maybe-x
        if (none? maybe-x) "nothing there";
        "the first is " ++ string (extract! maybe-x)
    );
    quit
```

> **Note.** The function `extract!` can crash, so use at your own risk.

Yet another alternative would be to use `?` and `map`. They're quite simple, but I'll explain them. Here are their types:

```funky
func ?   : a -> Maybe a -> a
func map : (a -> b) -> Maybe a -> Maybe b
```

The `?` function takes a _default value_ and a maybe and evaluates to the default value if the maybe is `none`. Otherwise it evaluates to the value inside the maybe. For example: `0 ? none` evaluates to `0`, while `0 ? some 3` evaluates to `3`.

The `map` function transforms the value inside a maybe using the supplied function. If the maybe is `none`, the result will be `none` as well. For example: `map (+ 1) none` results in `none`, while `map (+ 1) (some 3)` results in `some 4`. It works very much like `map` for lists.

We can use the `map` function to turn the `Maybe Int` obtained from `first nums` into a `Maybe String` like this:

```funky
map (\x "the first is " ++ string x) (first nums)
```

Then we can use `?` to fill the case when the list is empty:

```funky
"nothing there" ? map (\x "the first is " ++ string x) (first nums)
```

And here's the final program:

```funky
func main : IO =
    let [1, 2, 3, 4] \nums
    println ("nothing there" ? map (\x "the first is " ++ string x) (first nums));
    quit
```

That line is quite long. Too long!

But, there's a solution! The functions `?` and `map` have synonyms that are suitable for vertical code. Namely: `if-none` and `let-some`.

```funky
func if-none  : a -> Maybe a -> a
func let-some : Maybe a -> (a -> b) -> Maybe b
```

The function `if-none` is a precise synonym of `?`. And the only difference between `let-some` and `map` is the order of arguments.

We use `let-some` to safely extract a value from a maybe:

```funky
let-some (first nums) \x
"the first is " ++ string x
```

The whole thing becomes a new maybe. To turn it into a naked value, we handle the `none` case:

```funky
if-none "nothing there";
let-some (first nums) \x
"the first is " ++ string x
```

And here's the whole thing:

```funky
func main : IO =
    let [1, 2, 3, 4] \nums
    println (
        if-none "nothing there";
        let-some (first nums) \x
        "the first is " ++ string x
    );
    quit
```

That's more readable!

> **Note.** Of couse, there are many cases when `?` and `map` are the right choices. Perhaps this one is too. Depends on your personal preference.

In this short refactoring exercise, we've seen most of the useful `Maybe` functions: `none?`, `extract!`, `?`, `map`, `if-none`, `let-some`.

There are some more, like `some?`, `for-some`, `filter-some`, and `when-some`. You can try them out on your own.

## Thank you

You have reached the end of the tour. Thank you! I hope you find Funky interesting. It still needs a lot of work, but I'll do my best to make it as good as possible. If you'd like to get on that journey, you're more than welcome to join me on [TODO-CHAT-SERVICE](), or contribute on [GitHub](https://github.com/faiface/funky).
