---
title: Stabilizing Values 
date: 2015-04-08
tags: 
  - haskell
  - fixed points
---

Just recently I discovered a simple solution to an issue I encountered when
developing parts of my directed-cubical library. A very common pattern in the
code was to apply a function `f: a -> a` repeatedly to an initial value
`x` until the value stabilized, or in other words to find a *fixed point*.
The type signature of such a function looks something like

```haskell
stabilize :: (Eq a) => (a -> a) -> a -> a  
```

where the first argument is the function and the second argument is the initial
value. At first I figured that this must have something to do with our old
friend

```haskell
fix :: (a -> a) -> a
```

but the presence of an initial value suggests that we can't derive the answer
just by supplying some function. Recall that we use fix essentially by unfolding
the fixed point's value based on whatever we know about it already.  However, in
the case of "stabilize" we might be interested in a *strict* function `f`,
and there might be multiple fixed points of interest based on the choice of
initial point. Fortunately there's this lovely little function in the Haskell
Prelude:

```haskell
until :: (a -> Bool) -> (a -> a) -> a -> a
until p f = go
  where
    go x | p x          = x
         | otherwise    = go (f x)
 ```
 
Using this, we can define stabilize as follows:

```haskell
stabilize :: (Eq a) => (a -> a) -> a -> a
stabilize f x = until (\x -> x == f x) f x
```

This is nice and succinct, but observe that whenever we apply the predicate
`p` in the definition of `until` we compute `f x`, only to throw it away
and recompute it when recursing! Since `until` has no knowledge of our predicate
we end up evaluating `f x` values twice as often as we need to. However,
this points us in the right direction: what we really want is the list of *all*
iterates of `f` on the initial value `x` all at once, and this is
provided by our friend `iterate`:

```haskell
iterate :: (a -> a) -> a -> [a]
```

Given the list of all the iterates, we just need to pair each iterate with the
next (that's a zip!) and search for the first pair `(a,b)` where `a == b`,
but this is easy:

```haskell
stabilize' :: (Eq a) => (a -> a) -> a -> a
stabilize' f x = fst . head . dropWhile (uncurry (/=)) $
                 zip fxs (tail fxs)
   where fxs = iterate f x
```

This is a nice illustration of the utility of `uncurry`. The presence of `Eq`
is also important here: technically list types `[a]` shouldn't belong to this type
class because comparing infinite lists for equality doesn't terminate! (and this
easily results in problems in ghci). So `Eq a` is actually making a strong
assumption about our type `a`. We can partly recover `fix` with the following
definition:

```haskell
fix' :: (Eq a) => (a -> a) -> a
fix' f = stabilize' f (fix' f)
```

The `fix' f` part is us cheating to produce some element of `a` out of thin air.
Alas, to apply this we need a type `a` that is an honest instance of `Eq` as
well as supporting a least fixed point. I suppose infinite terms in a type with
`Eq` based on observable sharing would fit the bill, but I haven't gone down
that road yet.

