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

So, I guess you got it, right? **To define an an anonymous recursive function**, simply type `recur` and pass it a lambda where the first arguments acts as the name of the recursive function and the rest are its arguments. Simple as that.

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

So, with `|>`, we can rewrite the previos expression like this:

```funky
1 |> 0 |> recur \fibs \a \b
a :: fibs b (a + b)
```

This whole expression evaluates to just `[0, 1, 1, 2, 3, 5, 8, 13, 21, ...]`, the infinite Fibonacci sequence.

Now, since we're passing the arguments from the left, we need to pass them in the reverse order (actually, we're passing them in the correct order, but it reads as if it was reversed). I admit, this may all look kinda confusing to you, but don't worry, you'll get used it. It's very versatile, and the whole thing didn't require a single language addition.

### Back to reversing lists

So, how can we use `recur` to get rid of the extra `my-reverse-algo` function?

If you can, try figure it out yourself.

Here's how it looks with the extra function:

```funky
func my-reverse-algo : List a -> List a -> List a =
    \left \right
    if (empty? left) right;
    my-reverse-algo (rest! left) (first! left :: right)

func my-reverse : List a -> List a =
    \list
    my-reverse-algo list []
```

Here it is. All we have to do is replace `my-reverse-algo` in the body of `my-reverse` with an anonymous definition:

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

This is a little convention. In cases like this, we choose `loop` as the local name of the recursive function. It makes it immediately clear that we're using `recur` to simulate a simple stateful loop.