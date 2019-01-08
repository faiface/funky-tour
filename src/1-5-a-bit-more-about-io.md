# A bit more about IO

Funky has an unusual approach to side-effects. There's no notion of them in the language itself. What looked like side-effecting functions in the previous sections (`println`, `quit`, etc.) were in fact data structure constructors. The `IO` type is _not magic_ at all. It's just a plain data structure defined in the Funky language itself. Here's its definition (you can also find it in the standard library in the `cmd` folder):

```funky
union IO = quit | putc Char IO | getc (Char -> IO)
```

We haven't covered `union` yet but think algebraic data types (e.g. `data` in Haskell). This `IO` data structure serves as a description of what the program should do. However, in contrast with the control flow of traditional imperative languages, the `IO` data structure can be arbitrarily manipulated - e.g. it's no problem to implement input/ouput encryption just by implementing a single `IO -> IO` function.

> **Note.** The `IO` data structure is very limited. All it can express is printing and scanning characters. The thing we haven't talked about yet is that `funkycmd` is just one of the many possible _side-effect interpreters_. Specifically, `funkycmd` interprets (executes) the above `IO` data structure. There's another interpreter called `funkygame` that interprets an entirely different data structure which describes sprite-based games. You can actually make your own side-effect interpreter fairly easily. The specifics of that are explained in the [TODO]() section.

The `IO` looks like a linked list with two kinds of nodes (not counting `quit`). The `funkycmd` interpreter loads the `main` function, walks the `IO` nodes, and does as they say.

Let's talk about the nodes individually.

**Quit.** We've seen this one before - it marks the end of the program.

**Putc.** This node has two arguments: a `Char` and an `IO`. The `Char` is the character to be printed. The `IO` tells what should be done next.

**Getc.** Now this one looks strange. It has one argument: a function. The function takes a `Char` and returns an `IO`. Here's how `funkycmd` deals with `getc`: Upon encountering the `getc` node, `funkycmd` scans a character from the standard input. Then it takes the function under `getc` and applies it to the scanned character, passes it inside. The function gives back an `IO`. This `IO` tells what should be done next.

Now let's see them in action!

How about a very exciting program that reads a character and prints it back?

```funky
func main : IO =
    getc (\c putc c quit)
```

Run it:

```
$ funkycmd getcputc.fn
x
x
```

It worked!

Now, we can structure the code better using the same tricks we used with `if` and `let`. How does this look?

```funky
func main : IO =
    getc \c
    putc c;
    quit
```

Well, that's nice! The first line scans a character, the second line prints it back, the third line quits the program.

Wait, what would happen if we made it recursive? What if we replaced `quit` with `main`?

```funky
func main : IO =
    getc \c
    putc c;
    main
```

By the looks of it, this program should print back all of its input - it's the `cat` program.

> **Note.** But it never quits! Or does it? Currently, `funkycmd` is made to silently quit when encountering the EOF, so the program quits correctly. Of course, this behavior is not guaranteed to be kept.

```funky
$ funkycmd cat.fn
hello, cat!
hello, cat!
do you cat?
do you cat?
you do cat!
you do cat!
^D
```

The `^D` sequence at the end is the EOF.

## Scanning more than one character

The standard library gives us many goods. There's `print` and `println` that work by expanding to multiple `putc` nodes. Are there some analogous high-level functions for scanning? Sure there are! They're called, how unexpected, `scan` and `scanln`. What are their types:

```
$ funkycmd -types
> scan
(String -> IO) -> IO
> scanln
(String -> IO) -> IO
```

Oh nice, so their type is the same as the type of `getc`, except the `Char` is changed to `String`.

The `scanln` function scans a whole line and gives it back to you (not including the newline character). The `scan` function is a little more clever: it reads the next "word" from the input - it skips all the whitespace and captures a continuous string of non-whitespace characters.

We'll get to use `scan` more in the next section. Here's some `scanln` for the taste:

```funky
func main : IO =
    print "What's your name? ";
    scanln \name
    println ("Hello, " ++ name ++ "!");
    quit
```

And there we go:

```
$ funkycmd greeting.fn
What's your name? Michal Štrba
Hello, Michal Štrba!
```
