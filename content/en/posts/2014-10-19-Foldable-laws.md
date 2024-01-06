---
title: A Law for Foldable 
date: 2014-10-19
tags: 
 - semigroups
 - foldable
---

We are told by the documentation of
[Data.Foldable](http://hackage.haskell.org/package/base-4.7.0.1/docs/Data-Foldable.html)
that, at a minimum, a foldable instance must implement the foldMap function,
whose signature is

```haskell
foldMap :: Monoid m => (a -> m) -> t a -> m 
```

The fact that `m` is an arbitrary monoid allows us to determine the
underlying order in which elements are combined. To wit, consider the following
perfectly good foldable instances on lists:

```haskell
newtype FoldList a = FoldList [a]
instance Foldable FoldList 
   where foldMap f (FoldList as) 
      = Prelude.foldr (<>) mempty $ map f as

newtype FoldListRev a = FoldListRev [a]
instance Foldable FoldListRev
   where foldMap f (FoldListRev as) 
      = Prelude.foldr (<>) mempty $ map f (reverse as)
```

Then

```haskell
>> foldMap (\x -> [x]) $ FoldList [1,2,3,4]
[1,2,3,4]

>> foldMap (\x -> [x]) $ FoldListRev [1,2,3,4]
[4,3,2,1]
```

The upshot is that the type signature of foldMap actually *does* determine some
underlying traversal through `t`. The other way of defining a Foldable
instance is to specify foldr

```haskell
foldr :: (a -> b -> b) -> b -> t a -> b 
```

Similar to before, one can determine an order of traversal:

```haskell
>> Data.Foldable.foldr (\a b -> a:b) [] (FoldList [1,2,3,4])
[1,2,3,4]

>> Data.Foldable.foldr (\a b -> a:b) [] (FoldListRev [1,2,3,4]) 
[4,3,2,1]
```

A sane law to introduce for Foldable would therefore be:

```haskell
foldMap (\x -> [x]) ts == foldr (\a b -> a:b) [] ts
```

So foldable is not only about translating elements into monoid elements and
combining the results, but about *doing so in a specific order*. Fortunately if
you implement either foldMap or foldr but not both then (it appears) you can't
screw it up since the default implementation satisfies this law. For an example
of a Foldable instance that doesn't satisfy the law, consider

```haskell
instance Foldable ScrewyFoldable 
   where foldMap f (ScrewyFoldable xs) = mconcat $ map f xs 
         foldr f z (ScrewyFoldable xs) = Prelude.foldr f z (reverse xs)

>> foldMap (\x -> [x]) (ScrewyFoldable [1,2,3,4])
[1,2,3,4]

>> Data.Foldable.foldr (\a b -> a:b) [] (ScrewyFoldable [1,2,3,4])
[4,3,2,1]
```

It seems to me that it would be helpful to have this law stated in the
documentation of Foldable so that we could start to enforce it.

