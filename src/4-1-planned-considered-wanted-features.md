# Planned/considered/wanted features

## A package/module system _(wanted)_

The current idea is that identifiers prefixed with `_` will be unexported and importing would work similarly to Go's package system. All files in a single directory would be a part of a single package.

Also, a `record` with unexported fields wouldn't export its constructor and a `union` with unexported alternatives would not be switchable by the importer.

## More side-effect interpreters _(wanted)_

No side-effect interpreter is currently fully featured.

Funky aims to be a general purpose language without forgetting about visualizations, GUI, games, and recreational programming. Here are some of the most wanted side-effect interpreters:

- Command line applications with ability to replace shell scripts.
- GUI built for visualizations and games first, text editors second.
- Web servers.
- Erlang/Go-style concurrent distributed programs.

## Ability to combine multiple interpreters in a single program _(wanted)_

True power will come with the ability to use multiple side-effect interpreters in a single program. GUI & command line, concurrency & web, or all of them together. Not sure how to do it yet, but it's definitely something that wants to happen.

## The `with` clause _(planned)_

Currently, it's possible to make a `sum` function of this type:

```funky
func sum : List Int -> Int
```

Or this type:

```funky
func sum : List Float -> Float
```

And also other types that can be summed.

While the implementations will look very similar (or even the same), there's no way to generalize them. Just `List a -> a` won't work, because there's no way to add arbitrary types.

This is where the `with` clause comes handy:

```funky
with zero : a
with +    : a -> a -> a
func sum  : List a -> a = fold> (+) zero
```

This says that the `sum` function works on all types `a`, such that there are functions `zero` and `+` with the specified types.

The syntax is currently planned as shown - you need to use multiple `with` clauses to require multiple functions.

Another example would be:

```funky
with string : a -> String
func string : List a -> String =
    \list
    "[" ++ join ", " (map string list) ++ "]"
```

With regards to type collisions, the type of the `sum` function is `List a -> a`. If you attempted to also define a more specific version, like `sum : List Int -> Int`, it would fail with a type collision. This is a deliberate design choice to avoid having to select "the most specific" version and confuse everybody and instead have it simple: either it's general, or it's specific.

## Implicit functions _(considered)_

This feature would make a function be applied automatically when needed. This would usually be useful for automatic type-conversions.

An example:

```funky
implicit a -> Maybe a = some
```

The definition looks just like a definition of a normal function, except it starts with `implicit` instead of `func` and has no name.

This implicit function would enable writing this:

```funky
[1, 2, none, 4, 5, none, none]
```

instead of this:

```funky
[some 1, some 2, none, some 4, some 5, none, none]
```

The current idea is that an implicit function would only be applied when the original expression doesn't fit the context and only one implicit function could be applied to a single expression.

Also, defining an implicit function:

```funky
implicit A -> B = ...
```

would make types `A` and `B` collide.

So, for example, with the above `a -> Maybe a` conversion, these functions would collide even though they didn't before:

```funky
func int : String -> Int
func int : String -> Maybe Int
```

This is to avoid confusion and ambiguity.

This feature has a potential for abuse, that's why it's only considered and not planned yet.

## The `Box`/`Any` type _(planned)_

The `Box` type (could also be named `Any`) would be a built-in type with two built-in functions:

```funky
func pack    : a -> Box
func unpack! : Box -> a
```

The `unpack!` function could possibly have type `Box -> Maybe a` and be called `unpack`.

It would make it possible to temporarily escape the type system and allow creating some type-safe constructs not possible without it.

For example, suppose we'd like to make a side-effect interpreter for Erlang/Go-style concurrency. A natural way would be to define a `Routine` type, something like this:

```funky
union Routine a =
    halt                           |
    make (Chan a -> Routine a)     |
    send (Chan a) a (Routine a)    |
    recv (Chan a) (a -> Routine a) |
    spawn (Routine a) (Routine a)  |
```

This would enable expressing routines that make channels and send and receive values on them. The interpreter would handle actually sending values over the channels.

There's one crucial deficiency: all channels have to be of the same type. The current type system doesn't make it possible to express it any other way. You can try! It can't be done.

Obviously, we'd like to make channels of different types. Here's how the `Box` type would help:

```funky
union Routine =
    halt                       |
    _make (Box -> Routine)     |
    _send Box Box Routine      |
    _recv Box (Box -> Routine) |
    spawn Routine Routine      |
```

Three commands are prefixed with `_`. This means that they're not meant to be used by the user of this library and the future package system will make sure they're unexported.

The `Routine` type has also lost it's type variable. It doesn't need it anymore.

Now, this definition allows sending arbitrary values over arbitrary channels, but doesn't guarantee type-safety. However, if we define our own versions of those three commands, we get the type safety back:

```funky
record Chan a = _box : Box

func make : (Chan a -> Routine) -> Routine =
    \fnext
    _make \box-ch
    fnext (Chan box-ch)

func send : Chan a -> a -> Routine -> Routine =
    \ch \x \next
    _send (_box ch) (pack x);
    next

func recv : Chan a -> (a -> Routine) -> Routine =
    \ch \fnext
    _recv (_box ch) \box-x
    fnext (unpack! box-x)
```

These functions guarantee type-safe sending and receiving over channels and make it possible to create channels of different types.

## Partially apply second/third/... argument _(considered)_

Currently, we can only partially apply the first argument of a prefix function without making a lambda:

```funky
map sqrt
```

To partially apply a second argument, we need to make a lambda:

```funky
\f map f [1, 2, 3, 4, 5]
```

This syntax would make it possible to partially apply the second, third, or any other argument without making a lambda:

```funky
map _ [1, 2, 3, 4, 5]
```
