---
title: uncurry id == uncurry ($) 
date: 2015-07-26
tags: 
  - haskell
  - applicative
  - apply
  - maps
---

The other day I was trying to find if joining two `Map k` structures
together fell under a suitable abstraction. Following Conal Elliott, we start
from the fact that every `Map k a` can be regarded as a partial function `k -> a`
determined by a list of pairs `(k_i, a_i)`, and therefore as a total function
`k -> Maybe a`. But `Maybe a` is a monad and `k->` is a monad transformer to a Reader,
so certainly "`Map k`"s are monads, right?

Well, in Haskell the default `Monad` instance for `Maybe a` requires a `Monoid`
instance for `a`, so as long as we have that monoid instance then we are good.
However, what if we wanted to combine two maps more like this?

```haskell
Map k a -> Map k b -> Map k (a,b)
```

Here, pairing gives a monoidal product but not a `Monoid` object. Well, joining
Maps seems an awful lot like having an `Applicative`, in the sense that the above
looks like a monoidal product under f = Map k:

```haskell
f a -> f b -> f (a,b)
```

Then I stumbled upon the documentation for Kmett's semigroupoids library again,
and remembered that he says that `Map k` is an `Apply` but not an `Applicative`. The
typeclass `Apply` is just `Applicative` without "pure", and when I went to look at
the implementation it was just `Map.intersectionWith id`, where
`Map.intersectionWith` has type

```haskell
intersectionWith :: (a -> b -> c) -> Map k a -> Map k b -> Map k c
```

For normal people, this means that we are taking an inner join of the two maps
and combining the values at the common keys with the supplied binary function.
So what the hell does it mean to use the obviously *non-binary* function `id`?

Well, using `id :: a -> a` means that we have to unify `a ~ b -> c`, so we wind up
with

```haskell
intersectionWith id :: Map k (b -> c) -> Map k b -> Map k c
```

and at this point one can sort of look at the type, wave hands, and say that by
parametricity the only useful thing this function could do is apply each `(b -> c)` to the corresponding
`b` to get back a `c`. But how does the compiler know that this is the correct behavior? 

The definition of `intersectionWith` is not that interesting: it takes the `a -> b -> c` and applies it to 
each `a`, and then each `b`. If `a ~ b -> c`, then `id :: (b -> c) -> (b -> c)`, and the first step does nothing. 
The second step is the more interesting one: it applies the partally-evaluated function `b -> c` to `b` to yield
the `c` as expected. Uncurrying this step gives `uncurry ($): (b -> c, b) -> c`, which is the same as `uncurry id`!

So `id` is a binary operation after all... mind blown.  :-)

