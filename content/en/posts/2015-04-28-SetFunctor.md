---
title: Set is not a Functor (redux) 
date: 2015-04-28
tags: 
  - haskell
  - functors
---

Michael Snoyman has observed that [Set is not a
functor](https://www.fpcomplete.com/user/chad/snippets/random-code-snippets/set-is-not-a-functor). This is kind of
surprising if your initial hope is that `Set` is a `Monad`!

To the mathematically inclined: `Set` in `Haskell` represents not a category but rather a type constructor of kind 
`* -> *`. At a type `a`, `Set a` is the type of sets of elements all of type `a`. So, given a map `f: a -> b`, why
can't we get an induced map `fmap f: Set a -> Set b`?

The answer is that `a` and `b` here can be any types whatsoever, so in particular they do not require any notion of
equality of inhabitants! How, then, are we supposed to distinguish the set {\\(x_1, x_2\\)} from {\\(x_1\\)} where
\\(x_1, x_2\\) are just some inhabitants of `a`? Since we are not basing everything on set theory, we need to carry
around at least some kind of definition of equality for `a`, so at a minimum we need something like

```haskell
data Set a where
   Set :: (Eq a) => [a] -> Set a
```

In other words, a `Set a` implicitly carries around a context for equality of inhabitants of `a`. This is a problem for
putative functoriality of `Set` though, because there is *no reason whatsoever* for a map `f: a -> b` to respect the
`Eq` constraints on `a` and `b`! So this focuses our attention: we only want to consider maps `f: a -> b` that "respect"
these constraints, so that equals go to equals and our set formation works as expected.

Another point of view here is that our types `a` and `b`, together with the extra equality constraints, morally form
*setoids*: sets equipped with distinguished equivalence relations, or (as I prefer) groupoids consisting of contractible
path components. Then the statement should be something like: for every homomorphism `g: a -> b` of setoids, we
get a map of the corresponding `Set`s. 

Returning to Snoyman's example is enlightening: he constructs two isomorphic types, `a` and `AlwaysEq a`, with two
differing `Eq` constraints: in particular, `AlwaysEq a` decrees that all inhabitants are equal! The isomorphism is given by
a pair

```haskell
unAlwaysEq :: AlwaysEq a -> a
AlwaysEq :: a -> AlwaysEq
```

and we immediately observe that the former map is not a setoid homomorphism for any type `a` where there is at least one
pair of non-equal inhabitants.

What can we do to fix this? Well, something like this seems to work:

```haskell
instance (Show a) => Show (Set a) where show (Set as) = show $ nub as    

data Setoid a b where
   Setoid :: (Eq a, Eq b) => (a -> b) -> Setoid a b   

smap :: Setoid a b -> Set a -> Set b
smap (Setoid f) (Set as) = Set $ map f as 
smap (Setoid (+1)) (Set [1,2,3])
> [2,3,4] 
smap (Setoid (+1)) (Set [1,2,2,3])
> [2,3,4]
smap (Setoid (show)) (Set [1,2,2,3])
> ["1","2","3"]
```

We have introduced a type for setoid homomorphisms subject to `Eq` instances (necessarily unique) for `a` and `b`, which
type forms a Category. These preserve set formation so a functor obtains, just not a `Functor` on all of `Hask`.

