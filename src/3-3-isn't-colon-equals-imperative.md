# Isn't := imperative?

Sometimes, the best way to express a function is by a series of state-changing steps. In other words, by an _imperative algorithm_. For example, say we want to reverse a list. That's a fairly common thing. Sure, we could use the `reverse` function from the standard library, but how can we make one ourselves?

Here's a way to make it happen:

1. Have two variables: `left` and `right`.
2. Store the input list in the `left` variable.
3. Store an empty list in the `right` variable.
4. Repeat until the `left` list is empty:
    1. Add the first element of the `left` list to the beginning of the `right` list.
    2. Remove the first element from the `left` list.
5. Output the `right` list.

For example, let's try and reverse `[1, 2, 3, 4, 5]`. Here's how the algorithm would proceed:

```
           left | right
[1, 2, 3, 4, 5] | []
   [2, 3, 4, 5] | [1]
      [3, 4, 5] | [2, 1]
         [4, 5] | [3, 2, 1]
            [5] | [4, 3, 2, 1]
             [] | [5, 4, 3, 2, 1]
```

At the end, the `left` list is empty and the `right` list contains the reversed version of the original input list.

> **Note.** Adding/removing an element to/from the beginning of a list is efficient ([constant time](https://en.wikipedia.org/wiki/Time_complexity#Constant_time)). The whole algorithm therefore runs in time [linearly proportional](https://en.wikipedia.org/wiki/Time_complexity#Linear_time) to the size of the list.

How could we implement such an algorithm in Funky?

Well, this one isn't particularly hard. We can utilize recursion to simulate the state flow. We create a function called `my-reverse-algo`, which has two arguments: `left` and `right`. Those represent the two variables used in the algorithm. If the `left` list is empty, `my-reverse-algo` simply returns the `right` list. Otherwise, it returns from a recursive call with the variables updated appropriately.

```funky
func my-reverse-algo : List a -> List a -> List a =
    \left \right
    if (empty? left) right;
    my-reverse-algo (rest! left) (first! left :: right)
```

> **Details.** In most languages, each recursive call would grow the stack and we would quickly run out of memory. This doesn't happen in Funky thanks to [tail recursion](https://en.wikipedia.org/wiki/Tail_call). If a function should evaluate directly to another function call, it simply jumps to that function, effectively creating an efficient loop.

Now we'll use `my-reverse-algo` to implement `my-reverse`. It'll just pass the correct initial values to `my-reverse-algo`:

```funky
func my-reverse : List a -> List a =
    \list
    my-reverse-algo list []
```

Let's see if it works!

```funky
func main : IO =
    for (my-reverse [1, 2, 3, 4, 5])
        (println . string);
    quit
```

And run it:

```
$ funkycmd my-reverse.fn
5
4
3
2
1
```

Nice!

## The `recur` function

Could we somohow get rid of the helper `my-reverse-algo` function? It isn't really useful on its own, it's just an implementation detail. It shouldn't be there.

What would help would be some way to define recursive functions anonymously, i.e. without making a global `func` definition. That way we could define `my-reverse-algo` directly inside `my-reverse`.

The function for this job is called `recur`, otherwise known as the [_fixpoint combinator_](https://en.wikipedia.org/wiki/Fixed-point_combinator), or the _Y combinator_. It's a pretty mind-blowing function. I won't explain how it works (it took me a long time to figure it out), but I'll explain how you can use it.

Here's the type, but it won't help you figure out much:

```
$ funkycmd -types
> recur
(a -> a) -> a
```

Okay, so how to use it? Let's take this really simple recursive function:

```funky
func ones : List Int = 1 :: ones  # [1, 1, 1, ...]
```

It isn't even a function, it's just a recursive list.

We can express the same recursive list using `recur` like this:

```funky
recur \ones 1 :: ones
```

The variable `ones` isn't a global definition in this case, it's just a local bindning. Does it work?

```funky
func main : IO =
    for (take 5; recur \ones 1 :: ones)
        (println . string);
    quit
```

We only take the first five elements, because the list is infinite.

```
$ funkycmd ones.fn
1
1
1
1
1
```

It does work!

What about this function?

```funky
func count-from : Int -> List Int =
    \x x :: count-from (x + 1)
```

For example, `count-from 4` evaluates to `[4, 5, 6, 7, ...]`.

With `recur`, it looks like this:

```funky
recur \count-from \x
x :: count-from (x + 1)
```

You can try it yourself. Remember, that this time it's a function with one argument, so you need to pass it in.

So, I guess you got it, right? **To define an anonymous recursive function**, simply type `recur` and pass it a lambda where the first argument acts as the name of the recursive function and the rest are its arguments. Simple as that.

### The `|>` function

What about [Fibonacci](https://en.wikipedia.org/wiki/Fibonacci_number)?

```funky
recur \fibs \a \b
a :: fibs b (a + b)
```

This defines a function that given two initial numbers produces a Fibonacci-like sequence. If we give it 0 and 1 as the initial numbers, it will produce the Fibonacci sequence precisely:

```funky
(recur \fibs \a \b
a :: fibs b (a + b)) 0 1
```

Have you noticed them? The 0 and 1 that we passed in. They are right there, at the end of the expression. Perhaps you saw them, perhaps you didn't, but you surely must admit that it doesn't look very fashionable.

Instead of simply passing the arguments as usual, we can use the `|>` function to pass them. It works like this:

```funky
x |> f
```

is the same as:

```funky
f x
```

So, with `|>`, we can rewrite the previos expression like this, but we need to pass the arguments in a seemingly reverse order:

```funky
1 |> 0 |> recur \fibs \a \b
a :: fibs b (a + b)
```

> **Explanation.** The above code works, we pass `0` for `a` and `1` for `b`, not the other way around. This is because `y |> x |> f` is the same as `y |> (x |> f)`, which reduces to `y |> f x`, and finally `f x y`.

This whole expression evaluates to just `[0, 1, 1, 2, 3, 5, 8, 13, 21, ...]`, the infinite Fibonacci sequence.

I admit, the whole thing about passing the arguments in a seemingly reverse order may look kinda confusing to you, but don't worry, you'll get used it. It's very versatile, and the whole thing didn't require a single language addition.

### Back to reversing lists

Can we use `recur` to get rid of the extra `my-reverse-algo` function?

For the reminder, here's how it looks with the extra function:

```funky
func my-reverse-algo : List a -> List a -> List a =
    \left \right
    if (empty? left) right;
    my-reverse-algo (rest! left) (first! left :: right)

func my-reverse : List a -> List a =
    \list
    my-reverse-algo list []
```

And here's the solution. All we have to do is replace `my-reverse-algo` in the body of `my-reverse` with an anonymous definition:

```funky
func my-reverse : List a -> List a =
    \list
    (recur \my-reverse-algo \left \right
    if (empty? left) right;
    my-reverse-algo (rest! left) (first! left :: right)) list []
```

And we'll use `|>` to make it more readable:

```funky
func my-reverse : List a -> List a =
    \list
    [] |> list |> recur \my-reverse-algo \left \right
    if (empty? left) right;
    my-reverse-algo (rest! left) (first! left :: right)
```

**The last change we'll make** is we'll rename `my-reverse-algo` to simply `loop`:

```funky
func my-reverse : List a -> List a =
    \list
    [] |> list |> recur \loop \left \right
    if (empty? left) right;
    loop (rest! left) (first! left :: right)
```

**This is a little convention.** In cases like this, we choose `loop` as the local name of the recursive function. It makes it immediately clear that we're using `recur` to simulate a simple stateful loop.

## Procedures

Sometimes, simple loops like the above aren't powerful enough. Of course, we prefer them whenever possible. But some algorithms require a more nuanced expression.

A good example is the **imperative version of the [quicksort algorithm](https://en.wikipedia.org/wiki/Quicksort)**, i.e. quicksort on random-access arrays. That's a little too tough for the start, but we'll come back to it by the end of this part.

First, we need to learn how to express imperative algorithms. The standard library offers a type specifically for this purpose: `Proc`. That's short for _procedure_. It's a simple union with this definition:

```funky
union Proc s r =
    view (s -> Proc s r)       |
    update (s -> s) (Proc s r) |
    return r                   |
```

> **Note.** `Proc` is an equivalent of the [`State` monad](https://wiki.haskell.org/State_Monad) from Haskell and was initially called the same.

So, what's `Proc` all about? You can think of it as a description of a procedure that runs in some stateful context, (usually) changes it, and results in some return value. Let's take a close look!

First, it has **two type variables**:

- **`s`**. This is the type of the state context the procedure will run in. For example, if `s` is `Int`, then the procedure will begin its execution with some initial `Int`, say `4`, be able to change it and check its current value during its execution.
- **`r`**. This it the return type. When a procedure finishes, it returns some value, just like procedures in imperative languages.

Second, it's a union with **three alternatives or commands**:

- **`view`**. Used to check the current value of the state. Notice that its signature resembles the signature of `scanln` and similar functions. It's used the same way.
- **`update`**. This one is used to change the state. It takes a function of type `s -> s`. When executing the procedure, the current state will be mapped through this function. The second argument is a contination - tells what should be done next. The `update` function is used in the same way as `println` and similar functions.
- **`return`**. Marks the end of the procedure and specifies the return value.

Using these three instructions together with our already acquired battle-tested tools, like `if` and `for`, we can make descriptions of stateful procedures.

Let's see that in action!

Our first trivial example will be a procedure who's state context is just an `Int` and which returns an `Int` as well. When executed, it will increment the number in the state by 1 and return its new value.

```funky
func increment : Proc Int Int =
    update (+ 1);
    view \x
    return x
```

First, we update the state using `(+ 1)` (which is the same as `(\x x + 1)`). Then we get the current (updated) state and return it.

The **`return` function has an overloaded version**:

```funky
func return : (s -> r) -> Proc s r =
    \f
    view \x
    return (f x)
```

It takes the current state and makes a return value from it using the supplied function. In our case, we just want to return the state directly, so we'll use `self` (which is the identity function: `\x x`):

```funky
func increment : Proc Int Int =
    update (+ 1);
    return self
```

That's quite neat! Now, keep in mind, all we have constructed is a data structure - a description. How do we execute it?

**We need to provide some initial state value.** The function `start-with` is for just that. Here's how it looks like:

```funky
func start-with : s -> Proc s r -> r
```

It takes the initial state, then a procedure, and apparently it executes the procedure and evaluates to its return value. Let's see how that works!

```funky
func main : IO =
    let (start-with 6 increment) \x
    println (string x);
    quit
```

And run this:

```
$ funkycmd increment.fn
7
```

What if we want to execute `increment` multiple times? We can do that using the `call` function:

```funky
func call : Proc s a -> Proc s b -> Proc s b
```

This function is used for **executing one procedure inside another procedure**, discarding the return value of the first (we are able to get it, just wait a moment).

It takes two procedures and produces a new procedure that executes them one after another, returning the result of the second one.

```funky
func main : IO =
    let (start-with 6 (call increment increment)) \x
    println (string x);
    quit
```

Does it work?

```
$ funkycmd call.fn
8
```

Yep, it works! We can even reorganize it horizontally:

```funky
func main : IO =
    let (
        start-with 6;
        call increment;
        increment
    ) \x
    println (string x);
    quit
```

We could even increment four times!

```funky
func main : IO =
    let (
        start-with 6;
        call increment;
        call increment;
        call increment;
        increment
    ) \x
    println (string x);
    quit
```

The `call` function has another version with this type:

```funky
func call : Proc s a -> (a -> Proc s b) -> Proc s b
```

It's the same as the first version, except that instead of taking just a procedure as the second argument, it accepts a function taking the return value of the first procedure. The usage is quite straightforward.

Just for fun, let's gather all the return values from those three `increment`s using this new `call` version and return their sum:

```funky
func main : IO =
    let (
        start-with 6;
        call increment \x
        call increment \y
        call increment \z
        return (x + y + z)
    ) \w
    println (string w);
    quit
```

What will be the result?

```
$ funkycmd call.fn
24
```

That's because 7+8+9=24.

### Records as state contexts (`<-`, `->`, and `:=`)

Imperative algorithms usually work with **multiple variables**.

For example, a simple algorithm to accumulate an average of a stream of numbers must store two variables: the sum and the count. The final average is then obtained by dividing one by the other. To add a number to the average, we add it to the sum and we increase the count by 1.

A natural way to store these two variables in Funky is with a record:

```funky
record Average =
    sum   : Int,
    count : Int,
```

Then, using the record accessors, we can make a procedure to add a number to the average:

```funky
func add : Int -> Proc Average Nothing =
    \number
    update (sum (+ number));
    update (count (+ 1));
    return nothing
```

We use the `Nothing` return type, which is a type with only a single possible value: `nothing`. We use it to denote a procedure with no (useful) return value.

> **Note.** `Nothing` is the same type as `()` in Haskell or `Unit` in some other languages.

This isn't so bad. But, trust me, writing `update (field ...)` over and over gets tiresome with longer procedures. Records are so frequent here that the standard library provides some special treatment for them.

**The first function for this purpose is `<-`.** It's used to change a record field (or a nested field) in a procedure. Whenever you write this:

```funky
update (field function)
```

you can instead rewrite it to this:

```funky
field <- function
```

> **Note.** The `<-` function has type `((a -> a) -> s -> s) -> (a -> a) -> Proc s r -> Proc s r`.

Therefore, we can rewrite our `add` function into this beautiful piece of code:

```funky
func add : Int -> Proc Average Nothing =
    \number
    sum <- + number;
    count <- + 1;
    return nothing
```

Pretty cool, don't you think?

**Another useful function is `->`.** On the left side, it takes a getter, a function from the state type. Then it passes its result to the lambda on the right. It's like an enhanced `view` - whereas `view` always gives back the entire state, `->` gives you the part of the state you request.

For example, how about a procedure that returns the actual current average as an `Int`?

```funky
func average : Proc Average Int =
    sum -> \s
    count -> \c
    if (zero? c) (return 0);
    return (s / c)
```

Simple as that. Need to make sure to handle the case when the `count` is 0.

Using `add` and `average`, we can compose an imperative algorithm for calculating the average of a list:

```funky
func average : List Int -> Int =
    \numbers
    start-with (Average 0 0);
    for numbers (call . add);
    call average \avg  # call to the procedure, not a recursive call
    return avg
```

> **Note.** We overloaded the name `average` for two purposes here: one is the procedure and the other is the function. No worries, Funky can always tell the difference.

Notice that this `average` function has no `Proc` in the type. All the procedure stuff is encapsulated inside of it. To the outside world, it appears simply as another function.

**The last function for working with records in procedures is `:=`.** It's similar to `<-`, but instead of transforming the original value using a function, it simply overwrites it with a new value.

In other words, you can always replace this:

```funky
field <- const value
```

with this:

```funky
field := value
```

Say we wanted to reset the average counter. Here's what it would look like:

```funky
func reset : Proc Average Nothing =
    sum := 0;
    count := 0;
    return nothing
```

> **Details.** There's an overloaded version of `:=` which takes a function from the state instead of a direct value. It can be used to copy one part of the state into another. We could for example write: `sum := count`, even though that wouldn't make any sense. But, be careful. You **can't** write `sum := count * 2`, because `count` is a getter (a function) here, not a number. You could write `sum := (\s count s * 2)`, though.

By the way, **the function `self`** acts as both a getter and an updater for the whole state. Remember this function?

```funky
func increment : Proc Int Int =
    update (+ 1);
    return self
```

We can rewrite it:

```funky
func increment : Proc Int Int =
    self <- + 1;
    return self
```

Yes, this usage was one of the motivations for naming it `self` instead of `id`.

### Arrays

The last piece of puzzle before we can finally implement the quicksort algorithm is random-access arrays. 

Arrays in Funky are a bit different than arrays in imperative language, because we aren't allowed to mutate anything. To change an element in an array we must make a new array. But no full copies need to be performed. The new array can always share as much data as possible with the old one and thus the whole operation will be quite cheap.

> **Details.** Without cheating, there's really no such thing as in-place, mutable, [O(1)](https://en.wikipedia.org/wiki/Time_complexity#Constant_time) access arrays in pure functional programming. All data structures have to be primarily tree-based and persistent.
>
> However, there are efficient ways of implementing persistant arrays with O(log n) access time.

Currently, the `Array` type in Funky implements an infinite array indexed by `Int`s (both positive and negative). **To create an empty array, we use the `empty` function**:

```funky
func empty : a -> Array a
```

It takes one argument, which is the default initial value for all elements. For example:

```funky
empty 0
```

is an `Array Int` full of zeros. **We use `at` to both get and update any element** in the array. Here are the two overloaded versions:

```funky
func at : Int -> Array a -> a
func at : Int -> (a -> a) -> Array a -> Array a
```

As you have surely noticed, they look very similar to getters and updaters in records, except that they take an additional `Int` argument - the index. For example:

```funky
at 5 array
```

**gets the element at the index 5**, while:

```funky
at 5 (+ 3) array
```

evaluates to a **new array, where that element is increased by 3**.

There are two more functions, namely `reset` and `swap`. We won't cover the first one, but we'll come back to the second one when we get to the quicksort.

That's basically all there is to arrays. **The interesting part comes when integrating them with procedures.**

As we've already noticed, both version of `at` fit very well with the form of record accessors. And we can, in fact, use them the same way in procedures. For example:

```funky
func some-array : Array Int =
    start-with (empty 0);
    at 0 := 7;
    at 1 := 4;
    at 2 := 9;
    at 3 := 5;
    at 4 := 2;
    return self

func main : IO =
    for (range 0 4) (
        \i \next
        println (string; at i some-array);
        next
    );
    quit
```

We manually assign values to the elements of an array. Then, in `main`, we loop over the indices from 0 to 4 and print the corresponding elements from the array. Does it work?

```
$ funkycmd array.fn
7
4
9
5
2
```

Sure enough it does!

We can even shorten `some-array` with a for loop:

```funky
func some-array : Array Int =
    start-with (empty 0);
    for-pair (enumerate [7, 4, 9, 5, 2])
        (\i \x at i := x);
    return self
```

Now that we know arrays, let's get on to the quicksort!

### Quicksort

If you've ever implemented quicksort, you must know that it's quite tricky to get it right unless you're a super brilliant genius. At least the first time.

Because of that, we'll make our job easier and **simply translate [this algorithm from Wikipedia](https://en.wikipedia.org/wiki/Quicksort#Algorithm)**:

```
algorithm quicksort(A, lo, hi) is
    if lo < hi then
        p := partition(A, lo, hi)
        quicksort(A, lo, p - 1)
        quicksort(A, p + 1, hi)

algorithm partition(A, lo, hi) is
    pivot := A[hi]
    i := lo
    for j := lo to hi - 1 do
        if A[j] < pivot then
            swap A[i] with A[j]
            i := i + 1
    swap A[i] with A[hi]
    return i
```

That's a pseudocode. We'll try and translate it to Funky. We'll make two procedures: `quicksort` and `partition`.

Since these are imperative, we'll need the array in their mutable state. The `partition` procedure also uses an index `i`, so we'll also need that in the state. This will be our state:

```funky
record Vars =
    array : Array Int,
    i     : Int,
```

> **Note.** The `partition` procedure also uses an index `j`, but we need not include it in the state.

Okay, now let's try and translate `partition`. I won't do it step-by-step, this is more of a show-off than a tutorial, but I'm sure you'll be able to follow it. Here it is:

```funky
func partition : Int -> Int -> Proc Vars Int =
    \lo \hi
    (at hi . array) -> \pivot
    i := lo;
    for (range lo (hi - 1)) (
        \j \next
        (at j . array) -> \at-j
        when (at-j < pivot) (
            \next
            i -> \ii
            array <- swap ii j;
            i <- + 1;
            next
        );
        next
    );
    i -> \ii
    array <- swap ii hi;
    return ii
```

> **Details.** The `swap` function has type `Int -> Int -> Array a -> Array a` and does the obvious: evaluates to an array with the corresponding elements swapped.

It's definitely a little longer than the pseudocode. Most of the extra lines are from the lambdas and getting the current value of `i`. But otherwise, it reads very similar.

Now onto the `quicksort` procedure:

```funky
func quicksort : Int -> Int -> Proc Vars Nothing =
    \lo \hi
    if (lo >= hi)
        (return nothing);
    call (partition lo hi) \p
    call (quicksort lo (p - 1));
    call (quicksort (p + 1) hi);
    return nothing
```

This one is very straightforward. The only thing we changed from the pseudocode is that we inverted the condition. The pseudocode _does_ something when `lo < hi`, we instead _halt_ the procedure in the other case.

Let's see if this all works.

```funky
func main : IO =
    let (
        start-with (Vars (empty 0) 0);
        for-pair (enumerate [2, 6, 1, 0, 4, 3, 5, 9, 7, 8])
            (\i \x (array . at i) := x);
        call (quicksort 0 9);
        return array
    ) \arr
    for (range 0 9) (
        \i \next
        println (string; at i arr);
        next
    );
    quit
```

In this `main` function, we first start with an empty array and a zero `i` variable (which is irrelevant). Then we assign some random numbers to the first 10 indices of the array. Then we quicksort them and bind the resulting array to `arr` using `let`. Then we print the first 10 elements and see if it worked:

```funky
$ funkycmd quicksort.fn
0
1
2
3
4
5
6
7
8
9
```

It worked perfectly!

So, as you can see, we can write imperative algorithms quite straightforwardly in Funky!

## Mixing procedures with IO

Forget [monad transformers](https://en.wikibooks.org/wiki/Haskell/Monad_transformers) (if you've used Haskell), mixing procedures with IO isn't such a big deal. I'll show you how to do it.

So, we have this code:

```funky
func main : IO =
    let (
        start-with (Vars (empty 0) 0);
        for-pair (enumerate [2, 6, 1, 0, 4, 3, 5, 9, 7, 8])
            (\i \x (array . at i) := x);
        call (quicksort 0 9);
        return array
    ) \arr
    for (range 0 9) (
        \i \next
        println (string; at i arr);
        next
    );
    quit
```

But the `let` is in fact not needed. Remember, there's an overloaded version of `return` (which we are in fact using in this code) with this type:

```funky
func return : (s -> a) -> Proc s a
```

How about we used it rather differently than we already have? What if we passed it a function of type `Vars -> IO`? Something like this:

```funky
func main : IO =
    start-with (Vars (empty 0) 0);
    for-pair (enumerate [2, 6, 1, 0, 4, 3, 5, 9, 7, 8])
        (\i \x (array . at i) := x);
    call (quicksort 0 9);
    return \vars                              # <
    for (range 0 9) (                         # <
        \i \next                              # <
        println (string; at i (array vars));  # <
        next                                  # <
    );                                        # <
    quit                                      # <
```

I marked the function which we passed to `return`. It took `vars`, which is the whole state of the above procedure and made some `IO` out of it. The whole procedure now has type `Proc Vars IO` and so `start-with` turns it into just `IO`. Everything works:

```
$ funkycmd quicksort.fn
0
1
2
3
4
5
6
7
8
9
```

**That's how we switch from `Proc` to `IO`!** How do we switch back? Just `start-with` again! You have the last state, reuse it. We need to make sure, though, that we return some `IO` at the end.

Here's a simple program that reads 10 numbers from the input and prints out the average after each one (uses the `Average` record we defined earlier):

```funky
func main : IO =
    start-with (Average 0 0);

    for (range 1 10) (
        \i \next
        call average \avg
        return \a
        println ("Current average is " ++ string avg ++ ".");
        print ("Number #" ++ string i ++ ": ");
        scanln; int |> \x
        start-with a;
        call (add x);
        next
    );

    call average \avg
    return \a
    println ("Final average is " ++ string avg ++ ".");
    quit
```

I'm sure we could shave off some lines in a true imperative language, but so what! It's not bad for a purely functional one.

That's all for this part. In the next part, we'll shine some light on _"exception" handling_.
