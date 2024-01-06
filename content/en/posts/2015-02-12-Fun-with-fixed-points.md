---
title: Fun with fixed points 
date: 2015-02-12
tags: 
  - haskell
  - fixed points
---

If you import `Data.Function`, you'll find that the function `fix` has a funny
type:

```haskell
fix :: (a -> a) -> a
```

Given a function `f`, the value `fix f` will be "the" fixed point
of `f`. I think this means the *least* fixed point, which for some reason is
supposed to correspond to the *greatest* fixed point in Haskell, but I've still
not quite understood enough domain theory to explain why. If \\(f\\) is a strict
function then \\(f \bot = \bot\\), so least fixed points of strict functions are
easy to understand. 

The definition of fix looks like this:

```haskell
fix f = let x = f x in x
```

So in some sense we start with some `x` (which one?) and apply `f`
infinitely many times to it. This does not give us a terribly good operational
understanding of how we can compute this thing. In fact, it is easier to
understand `fix f` in terms of its key property:

```haskell
fix f = f (fix f)
```

In words, applying `f` to its own (least) fixed point is going to give you
back that point! Now suppose we start with some lazy `f` like
`const 1`, the constant function \\(\lambda x. x \to 1\\). Then

```haskell
fix (const 1) = (const 1) (fix (const 1))
              = 1
```

thanks to lazy evaluation, so we have a fixed point that isn't \\(\bot\\). Yay!

Looking at the equation above again, we see that we won't be able to make
progress with computing `fix f` unless `f` can be at least partially
defined without referring back to `fix f`. This suggests the following
example:

```haskell
let fibs = fix (\p -> 1 : 1 : zipWith (+) p (tail p))
```

This is the infinite list of fibonacci numbers, usually written without using
`fix`. The function we use takes it own fixed point as an argument, and then
provides some partial information about its value before tying the knot by using
its fixed point again. Given a list of integers, this function produces a new
list of integers starting with 1, 1. Plugging *any* list starting with 1, 1 back
into the function yields a list starting with 1, 1, 2, because we know how to
compute at least the next term at this point. At each step, we learn a bit more
information about the fixed point `p`, and we find that it is the fibonacci
sequence itself. This looks like a kind of corecursion.

It's slightly more surprising that we can also use this trick to implement
certain types of *recursion*. I'll use the other example, which is the factorial
function:

```haskell
let fact = fix (\f n -> if n == 0 then 1 else n * f (n-1))
```

The argument to `fix` in this case is a function of type

```haskell
(Integer -> Integer) -> (Integer -> Integer)
```

so the fixed point will be a *function* rather than just a value! Of course, all
functions *are* just values inhabiting appropriate function spaces, so it should
not be so surprising that we can do this. Again, we start with some partial
information about the fixed point (its behavior at 0 in this case), and then
"pump" it for more information as we need it. The underlying function

```haskell
g = \f n -> if n == 0 then 1 else n * f (n - 1)
```

is *not* strict, since 

```haskell
g undefined = \n -> if n == 0 then 1 else undefined
```

which is a perfectly good non-\\(\bot\\) function! There is clearly a whole
world of interesting behavior outside of strict functions.

Now: what if we did this at the type level rather than the term level? Then we
are talking about fixed points of type constructors... hmm!

