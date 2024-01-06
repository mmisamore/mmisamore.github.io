---
title: The commutative monoid of the Maybe monad 
date: 2014-06-27
tags: 
  - monads
  - monoids
---

The `Maybe` monad is a certain endofunctor on the category of types. There
is another endofunctor called `Id` on this category, and a natural
transformation `Just: Id -> Maybe`. For any object `a` there is
therefore a morphism `Just: a -> Maybe a`. There is another endofunctor,
namely `*`, sending every type to the unit type `()`, and a
natural transformation `Nothing: * -> Maybe`.

To describe the monoid underlying `Maybe` it suffices to define the natural
transformation \\(\mu: Maybe \circ Maybe \to Maybe\\) via precompositions with
`Just` and `Nothing` on either side. On objects we must therefore define
maps \\(\mu_a: Maybe\\,(Maybe\\,a) \to Maybe\\,a\\). 

Consider the composite 

$$a \xrightarrow{Just} Maybe\\,a \xrightarrow{Just} Maybe\\,(Maybe\\,a)
\xrightarrow{\mu_a} Maybe\\,a,$$ 

and observe that this equals \\(Just\\,a\\) by definition of \\(Maybe\\). 
Similarly, in the composites

$$a \xrightarrow{Just} Maybe\\,a \to () \xrightarrow{Nothing} Maybe\\,(Maybe\\,a) 
\xrightarrow{\mu_a} Maybe\\,a$$

$$() \xrightarrow{Nothing} Maybe\\,a \xrightarrow{Just}
Maybe\\,(Maybe\\,a) \xrightarrow{\mu_a} Maybe\\,a$$

$$() \xrightarrow{Nothing} Maybe\\,a \to () \xrightarrow{Nothing}
Maybe\\,(Maybe\\,a) \xrightarrow{\mu_a} Maybe\\,a$$

the values all factor through \\(Nothing\\), which serve to define \\(\mu_a\\)
over the possible (non-bottom) inhabitants of \\(Maybe\\,(Maybe\\,a)\\).

Colloquially this has a much simpler description: it is the monoid with a single
non-trivial idempotent \\(x\\). That is, there is an identity element \\(1\\)
represented by \\(Just\\), and an idempotent element \\(x\\) represented by
\\(Nothing\\). It is automatically a commutative monoid since there is only one
non-identity element (again discounting \\(\bot\\)).

Some have argued that commutativity can fail if one allows bottom: 

```haskell
do undefined; Nothing
>> *** Exception: Prelude.undefined
```

vs.

```haskell
do Nothing; undefined
>> Nothing
```

However, this code essentially desugars down to

```haskell
undefined >>= \x -> Nothing
```
    
```haskell
Nothing >>= \x -> undefined
```

where the former case is 

```haskell
join . fmap (\x -> Nothing) $ (undefined :: Maybe a)
```

and the latter is

```haskell
join . fmap (\x -> undefined) $ Nothing
```

Crucially, the definition of `>>=` here is using `fmap` in addition to `join`, and
`fmap` is strict in its arguments which results in the failure. What about using
`join` without `fmap`?

```haskell
let t x = (); t :: a -> ()
let g x = Nothing; g :: () -> Maybe a
let nothing = g . t

join $ nothing (undefined :: Maybe a)
>> Nothing

join $ Just Nothing
>> Nothing
```

Here `nothing` has type \\(Maybe\\,a \to Maybe\\,(Maybe\\,a)\\), so it appears
that `Maybe` is commutative after all, at least in GHC's implementation of `join`. 

