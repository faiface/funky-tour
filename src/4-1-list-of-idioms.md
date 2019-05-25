# List of idioms

This is an article that lists most of the idioms frequently used in Funky code. Each idiom is introduced by a name with a link to the place in this tour that describes it, a short description and some schematic code examples.

## Naming

- Dashes to separate words.
  ```funky
  yield-all right-pad format-table Priority-Queue
  ```
- Question marks at the end of predicates (functions that return `Bool`):
  ```funky
  even? odd? whitespace? empty? none? some?
  ```
- Exclamation marks at the end of partial functions (functions that can crash!):
  ```funky
  first! rest! extract! at!
  ```
- Special symbols where appropriate:
  ```funky
  + - * / abs^2 let-::
  ```

## [Vertical functions](1-4-semicolons-and-lambdas.md)

These are the building block of code that reads top to bottom. The example below show some, but not all of the common patterns of types these functions tend to have.

```funky
func semicolon-based-1 : V -> V =
    ...

func semicolon-based-2 : A -> V -> V =
    ...

func lambda-based-1 : (B -> V) -> V =
    ...

func lambda-based-2 : C -> (D -> V) -> V =
    ...

func lambda-based-3 : E -> (F -> G -> V) -> V =
    ...

func lambda-based-4 : I -> (J -> V -> V) -> V -> V =
    ...

func usage : V =
    semicolon-based-1;
    semicolon-based-2 a;
    lambda-based-1 \b
    lambda-based-2 c \d
    lambda-based-3 e \f \g
    lambda-based-4 i (
        \j \next
        ...
        next
    );
    ...
```

## [If](1-4-semicolons-and-lambdas.md#the-if-function)

The `if` function is used to choose from two alternatives based on a condition.

### Conditional expression (or the ternary operator)

```funky
println (if condition then else);
println (if (x < y) x y);
...
```

### Short if/else

```funky
if (n <= 0) 1;
n * factorial (n - 1)
```

```funky
if (invalid? input)
    (println "invalid"; quit);
println "correct!";
...
```

### Long if/else

```funky
if (invalid? input) (
    println "your input is wrong!!!";
    println "now you will suffer.";
    ...
);
...
```

### If/else chain

```funky
if (op == "+") ...;
if (op == "-") ...;
if (op == "*") ...;
if (op == "/") ...;
if (op == "^") ...;
"invalid operator"
```

## [When](3-2-yield-it-all.md#the-when-function)

The `when` function is used to conditionally apply/execute a function unlike `if` which is used to choose between two alternative branches. The imperative analog of `when` is an `if` without an `else`.

### Short when

```funky
when condition function;
...
```

```funky
when (total-price cart < euro 30.0)
    (println "The shipping will not be free.");
...
```

```funky
when (p x) (x ::) (filter xs)
```

### Long when

```funky
when condition (
    \next
    ...
    next
);
...
```

## [Let](1-4-semicolons-and-lambdas.md#the-let-function)

Binding values to variables is not a built-in construct in Funky.

### Single binding

```funky
let (reverse; range 1 1000000) \numbers
...
```

### Multiple bindings

```funky
let (filter (< pivot) nums)  \left
let (filter (== pivot) nums) \middle
let (filter (> pivot) nums)  \right
...
```

### Pair binding (not mentioned in the tour)

```funky
let-pair (pair "A" 4) \s \x
...
```

### Long binding

```funky
let (
    ...
) \variable
...
```

### [Maybe binding](3-4-on-being-exceptional.md)

Both of the expressions have type `Maybe Int`.

```funky
let-some (some 7) \x
x + 10
```

```funky
let-some none \x
x + 10
```

### [List binding](3-4-on-being-exceptional.md)

Both of the expressions have type `Maybe something` (`something` is just a placeholder depending on the type of `...`).

```funky
let-:: [1, 2, 3, 4, 5] \x \xs  # x=1 and xs=[2,3,4,5]
...
```

```funky
let-:: [] \x \xs  # the whole expression is none
...
```

## [For](3-1-for-loop-is-a-function.md)

The `for` function is used to loop over a list of elements. It is functionally equivalent to `fold<`.

### [Short for loop](3-1-for-loop-is-a-function.md#how-to-print-a-list-meet-the-for-loop)

```funky
for list function;
...
```

```funky
for ["A", "B", "C"] println;
quit
```

```funky
for (range 1 10)
    (println . string);
quit
```

### [Long for loop](3-1-for-loop-is-a-function.md#how-to-put-more-things-in-a-body)

```funky
for list (
    \x \next
    ...
    next
);
...
```

### [Looping over pairs](3-2-yield-it-all.md#formatting-lists)

```funky
for-pair list-of-pairs (
    \x \y \next
    ...
    next
);
...
```

```funky
for-pair (enumerate names) (
    \i \name \next
    println ("#" ++ string i ++ ": " ++ name);
    next
);
quit
```
