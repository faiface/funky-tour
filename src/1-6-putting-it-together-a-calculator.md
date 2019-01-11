# Putting it together: a calculator

We've covered a few basic functions, now we'll put them to the test! We'll make a little calculator. The program is simple but nicely demonstrates how all the functions we've talked about work _together_. Let's see!

The calculator will work something like this:

```
$ funkycmd calculator.fn
2 + 2
4
9 / 2
4.5
12 * 12
144
```

It will support four operators: `+`, `-`, `*`, `/`. It will only support one operator per expression.

The first thing will be to scan the expression. The expression consists of three words on a single line, so it'll be best to use `scan` three times:

```funky
func main : IO =
    scan \x-str
    scan \op
    scan \y-str
    # TODO
```

Good. Now, `x-str` and `y-str` are strings, because that's what `scan` gives us. In order to perform computations, we need to convert them to floats first.

```
$ funkycmd -types
> float
Int -> Float
String -> Float
```

The second overload of the `float` function seems to be doing just that, so let's use it (there's an analogous function for integers called `int`):

```funky
func main : IO =
    scan \x-str
    scan \op
    scan \y-str
    let (float x-str) \x
    let (float y-str) \y
    # TODO
```

We saved the converted numbers in the `x` and `y` variables.

Now we need to compute the result and print it. For example, if the operator is `+`, the result will be `string; x + y`. Here we can exploit the `if` function's ability to form if/else chains:

```funky
if (op == "+") (string; x + y);
if (op == "-") (string; x - y);
if (op == "*") (string; x * y);
if (op == "/") (string; x / y);
"invalid operator: " ++ op
```

The last line becomes the result if the operator is not among the supported ones. Now that we've got the result, all that's left it to print it and continue the loop:

```funky
func main : IO =
    scan \x-str
    scan \op
    scan \y-str
    let (float x-str) \x
    let (float y-str) \y
    println (
        if (op == "+") (string; x + y);
        if (op == "-") (string; x - y);
        if (op == "*") (string; x * y);
        if (op == "/") (string; x / y);
        "invalid operator: " ++ op
    );
    main
```

**And that's it!** Now go and experiment yourself! When you come back, we'll start learning about making our own functions and types.
