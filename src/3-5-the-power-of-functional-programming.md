# The power of functional programming

Over the course of the last few parts, we've learned how to do for-loops, Python-like generators, and imperative algorithms. What has that got to do with functional programming?

Someone once said: _Functional programming makes difficult tasks easy and easy tasks difficult._

What they meant was that while it's often easy to express a complex algorithm in a few (or even one) lines of code, common things, like making a simple interacting command line program can be challenging and not elegant at all.

I believe this is because some problems are just easier to express imperatively. I thus hope that by making imperative idioms easily expressible in Funky, we can have a language where difficult tasks are easy and easy tasks are easy alike. Really difficult tasks will remain difficult, I'm sorry.

So, how does Funky support the _proper_ functional programming idioms? In this part, we'll focus on those, particularly lists: the powerhouse of functional programming, and take a look at some really cool programs.

## Pascal's triangle

Our first problem will be to construct the [Pascal's triangle](https://en.wikipedia.org/wiki/Pascal%27s_triangle) as it is called, although it was definitely not first discovered by [Blaise Pascal](https://en.wikipedia.org/wiki/Blaise_Pascal).

It looks like this:

```
              1
            1   1
          1   2   1
        1   3   3   1
      1   4   6   4   1
    1   5   10  10  5   1
  1   6   15  20  15  6   1
1   7   21  35  35  21  7   1
```

It's all ones on the edges, but other than that, each number is exactly the sum of the two numbers above it. Those are just the first eight rows, of course, there's infinite number of them. If you're not familiar with the properties of this triangle, I highly recommend checking out the [Wikipedia page](https://en.wikipedia.org/wiki/Pascal%27s_triangle), they're quite remarkable.

Alright, so let's make Pascal's triangle. Since Funky is lazy, we can express it as an infinite list of rows. Each row of course is a list of integers:

```funky
func pascal's-triangle : List (List Int) =
    # ???
```

Now, the first row is `[1]`. It would be nice if we make some function that can transform one row into the next one. Then we could use `iterate`, which works like this:

```funky
iterate f x = [x, f x, f (f x), f (f (f x)), ...]
```

to generate the whole list of rows.

```funky
func pascal's-triangle : List (List Int) =
    iterate ??? [1]
```

What could be the function to transition from one row to the other? The function `adjacent` comes handy here. It works like this:

```funky
adjacent f [x, y, z, w, ...] = [f x y, f y z, f z w, ...]
```

It pairs each pair of adjacent elements in the list with the supplied function. That's almost exactly what we want. For example:

```funky
adjacent (+) [1, 3, 3, 1] => [4, 6, 4]
```

All that's missing are the ones at the edges. That's easy to fix:

```funky
[1] ++ adjacent (+) [1, 3, 3, 1] ++ [1] => [1, 4, 6, 4, 1]
```

And here's the final function:

```funky
func pascal's-triangle : List (List Int) =
    iterate (\row [1] ++ adjacent (+) row ++ [1]) [1]
```

That's really elegant! Let's move on to another example.

## Quick-sort

You may have seen this one, but I have included it anyway, because it's so nice. The quick-sort algorithm is a sorting algorithm that works like this:

1. We pick a single element out of the list, we call it a _pivot_.
2. We divide the list into three groups: _less than_ pivot, _more than_ pivot, _equal to_ pivot.
3. We know the order in which these groups should appear in the final sorted list, so we sort the first two recursively, then put everything together.

Here, we'll use the first element as the pivot for simplicity. The best choice would be the median, [which can be found in O(n) time](https://en.wikipedia.org/wiki/Median_of_medians). Choosing the first element may result in O(n^2) time for some lists, especially nearly sorted ones. However, it will be optimal for randomly ordered lists.

The algorithm as described above can be translated to code nearly exactly (we use `List Int` for simplicity again, but this can be done generally by taking the `<` function as another argument):

```funky
func quick-sort : List Int -> List Int =
    \nums
    if (empty? nums) [];
    let (first! nums)            \pivot
    let (filter (< pivot) nums)  \less
    let (filter (== pivot) nums) \equal
    let (filter (> pivot) nums)  \more
    quick-sort less ++ equal ++ quick-sort more
```

And that's it! The only little code smell is the use of `first!`, but that can be easily fixed:

```funky
func quick-sort : List Int -> List Int =
    \nums
    if-none [];
    let-some (first nums)        \pivot
    let (filter (< pivot) nums)  \less
    let (filter (== pivot) nums) \equal
    let (filter (> pivot) nums)  \more
    quick-sort less ++ equal ++ quick-sort more
```

Now it's perfect.

## Integrating

Calculating the [derivative](https://en.wikipedia.org/wiki/Derivative) of a function is quite easy:

```funky
func differentiate : (Float -> Float) -> Float -> Float =
    \f \x
    let 1e-8 \eps
    (f (x + eps) - f (x - eps)) / (2.0 * eps)
```

Then, we can use it:

```funky
differentiate (\x x ^ 2.0)
```

The above results in a function that's almost equal to:

```funky
\x 2.0 * x
```

As it should be.

So, how about calculating the [integral](https://en.wikipedia.org/wiki/Integral)? A function like:

```funky
func integrate : (Float -> Float) -> Float -> Float =
    # ???
```

would be really nice.

We are gonna use the standard method to compute the integral, which is to:

1. Split the interval [0, x] into many equally-sized chunks.
2. Create a rectangle above (under) each of the chunks who's height matches the value of the function at that point.
3. Calculate the area of each of the rectangles.
4. Sum up those areas.

If you don't understand the algorithm, consider watching [this video](https://www.youtube.com/watch?v=dEAk0BHBYCM).

We're gonna express the algorithm using the method of _threading_. Threading means passing an initial value through a series of transformations. It can be expressed very beautifully using the `|>` operator. As we know, the `|>` operator works like this:

```funky
(x |> f) = (f x)
```

Now, to pass a value through a series of functions, we would like to write something like this:

```funky
x |> f |> g
```

But, this would get parenthesised as:

```funky
x |> (f |> g)
```

which is not what we want.

However, if you type the expression `x |> f |> g` in Funky, you find that it actually works! This is because `|>` has another version that works like the function composition operator:

```funky
(f |> g) = (g . f)
```

When threading, we usually write the code like this, for readability:

```funky
x
|> f
|> g
```

Now, let's get to the integral. First, we're gonna specify the width of each of the rectangles:

```funky
func integrate : (Float -> Float) -> Float -> Float =
    \f \x
    let 0.001 \w
    # ...
```

The width we chose isn't very small, this is for performance reasons: a very small width would result in too many rectangles.

Now we're gonna generate the positions of the individual rectangles:

```funky
func integrate : (Float -> Float) -> Float -> Float =
    \f \x
    let 0.001 \w
    iterate (+ w) 0.0
    |> take-while (<= x)
    # ...
```

We generate an infinite list of `w`-separated numbers starting from zero, then we cut them at `x`. Now, we're gonna transform this list into the values of the function at those points:

```funky
func integrate : (Float -> Float) -> Float -> Float =
    \f \x
    let 0.001 \w
    iterate (+ w) 0.0
    |> take-while (<= x)
    |> map f
    # ...
```

Now we're gonna calculate the areas:

```funky
func integrate : (Float -> Float) -> Float -> Float =
    \f \x
    let 0.001 \w
    iterate (+ w) 0.0
    |> take-while (<= x)
    |> map f
    |> map (* w)
    # ...
```

And finally, sum them all up:

```funky
func integrate : (Float -> Float) -> Float -> Float =
    \f \x
    let 0.001 \w
    iterate (+ w) 0.0
    |> take-while (<= x)
    |> map f
    |> map (* w)
    |> sum
```

And that's all! The threading pattern is useful in many situations and usually results in a very readable code, so use it when appropriate.

## Thank you

You have reached the end of the tour. Thank you! I hope you find Funky interesting. It still needs a lot of work, but I'll do my best to make it as good as possible. If you'd like to get on that journey, you're more than welcome to join me on [TODO-CHAT-SERVICE](), or contribute on [GitHub](https://github.com/faiface/funky).
