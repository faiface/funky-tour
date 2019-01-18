# Records

Now onto creating our own types!

I'm sure you're familiar with `struct`s from C or `record`s from Pascal. They simply group multiple values into a single value. Records in Funky are very much the same.

They're defined like this:

```funky
record Name = field : type, ...
```

For example:

```funky
record Person =
    name : String,
    age  : Int,
```

> **Note.** Trailing comma is allowed.

The `Person` type is now a new, non-reproductive way of creating people (pretty revolutionary if you think about it). How do we create one?

Whenever we define a `record`, Funky **generates a constructor function with the same name**. Let's see!

```
$ funkycmd -types person.fn
> Person
String -> Int -> Person
```

Basically, it takes values for the name and the age and gives us a fully fledged person. Great!

```funky
record Person =
    name : String,
    age  : Int,

func main : IO =
    let (Person "Joe" 28) \joe
    quit
```

We've got Joe! How do we use him?

In addition to the constructor function, Funky **generates two functions per field: a getter and an updater**. Name of both of the functions is the same as the name of the field:

```
$ funkycmd -types person.fn
> name
Person -> String
(String -> String) -> Person -> Person
```

The type `Person -> String` is the getter. It works just as you'd expect:

```funky
record Person =
    name : String,
    age  : Int,

func main : IO =
    let (Person "Joe" 28) \joe
    println (name joe);
    quit
```

Running it:

```
$ funkycmd person.fn
Joe
```

The updater with the type `(String -> String) -> Person -> Person` is a little more... funky.

Here's what it does: it takes a function and a person. It returns a new person that is the same as the old one, except with the field modified by the function.

Let's see:

```funky
record Person =
    name : String,
    age  : Int,

func main : IO =
    let (Person "Joe" 28)       \joe
    let (name (++ ", yo!") joe) \joe  # shadows the previous joe variable
    println (name joe);
    let (age (+ 1) joe) \joe          # Joe had a birthday!
    println (string; age joe);
    quit
```

And run it:

```
$ funkycmd person.fn
Joe, yo!
29
```

> **Note.** If you want to replace the field's value by some other value, you can use the `const` function like this: `name (const "Joseph") joe` (changes the name to `"Joseph"`). The function `const` makes a function that, taking any input, always evaluates to a constant. So, `(const "Joseph") 12` evaluates to `"Joseph"`.

## Composing getters and updaters

The signatures of the getters and the updaters are not random. They enable some really cool stuff! We'll check it out now.

I'll show it to you on these two records:

```funky
record Point =
    x : Int,
    y : Int,

record Line =
    from : Point,
    to   : Point,
```

Okay, now, say we have a `Point`, let's call it `pt`. To get the X coordinate we do `x pt` and to get the Y coordinate we do `y pt`. Easy! To move the X coordinate 10 units to the right, we do `x (+ 10) pt` and to set the Y coordinate to 0 we do `y (const 0) pt`.

Okay, now say we have `Line`. Let's call it `line`. The `line` has two points: `from` and `to`. What if we want to move the X coordinate of the first point by 10?

Now, here comes the cool part! All we need to do it to realize, that

```funky
x (+ 10)
```

is a partial application of the `x` updater and has type `Point -> Point`. But, any `Point -> Point` can be used in the `from` updater:

```funky
from (x (+ 10)) line
```

Lastly, we can clean it up with the function composition operator:

```funky
(from . x) (+ 10) line
```

> **Note.** The function composition operator `.` works like this: `(f . g) x = f (g x)`. So you see we've just rewritten the previous expression using this formula.

> **Details.** In Funky, the function composition operator is overloaded a few times in the standard library, namely for when the second function takes multiple arguments. For example, one of the overloaded versions works like this: `(f . g) x y = f (g x y)`. That way, writing `(concat . map)` works exactly as you'd like to. The type checker can always tell which version fits the context.

And that's it! That's pretty clean, isn't it?

Getters can be composed similarly, except in the opposite order:

```funky
(y . to) line
```

That's the Y coordinate of the last point.

Getters and updaters also have a very important role when it comes to expressing state flow with the `State` type as we'll cover in the [TODO]() part. In short, they enable record fields to act as mutable variables in an imperative algorithm.
