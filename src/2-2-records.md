# Records

Now onto creating our own types!

I'm sure you're familiar with `struct`s from C or `record`s from Pascal. They simply group multiple values into a single value. Records in Funky are very much the same.

They're defined like this:

```funky
record Name = field-a : type-a, field-b : type-b, ...
```

For example:

```funky
record Person =
    name : String,
    age  : Int,
```

> **Note.** Trailing comma is allowed.

The `Person` type is now a new, non-reproductive way of creating people (pretty revolutionary if you think about it). How do we create a person?

Whenever we define a `record`, Funky automatically **generates a constructor function with the same name**. Let's see!

```
$ funkycmd -types person.fn
> Person
String -> Int -> Person
```

Basically, it takes the value for the name and the age and gives us a fully fledged person. Great!

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

> **Note.** If you want to replace the field's value by some other value, you can use the `const` function like this: `name (const "Joseph") joe` (changes the name to `"Joseph"`).

## Composing getters and updaters

