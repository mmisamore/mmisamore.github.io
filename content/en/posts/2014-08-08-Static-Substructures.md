---
title: Statically Typed Substructures 
date: 2014-08-08
tags: 
  - static typing
  - data
---

Suppose I want to write a function that takes a list together with one of its
elements, and produces a new list of the same type. Here is a naive attempt at a
type signature:

```haskell
f :: [a] -> a -> [a]
```

Do you see the problem? There is no guarantee that the second argument is
actually an element of the list, so

```haskell
f [1,2,3] 4
```

should not be a valid invocation of the function. One way to fix this might be
to make `f` a no-op when the element `a` does not belong to the input list, but
this allows for future bugs: the function may *silently fail* when the caller
expects it to succeed!

A better idea is to introduce a data structure that statically encodes the
invariant that the element belongs to the list:

```haskell
data EltInList a = EltInList { _inList :: [a], _point :: a } 
  deriving (Show,Eq)
eltOf :: EltInList a -> a

listElts :: [a] -> [EltInList a]
listElts l = zipWith EltInList (repeat l) l
```

If this type is made opaque in the API then we can now write our function with
signature

```haskell
f :: EltInList a -> [a]
```

which *statically* enforces the invariant of a substructure (in this case an
element) belonging to a larger structure. What is the pattern for other
container types? Well, suppose the problem is to implement functions like

```haskell
f :: g a -> a -> g a
```

while ensuring that the second argument belongs to the container of type `g a`.

```haskell
data EltIn g a = EltIn { _in :: g a, _point :: a }
  deriving (Show,Eq)
```

To implement `listElts`, it is certainly enough to have a `Foldable g`:

```haskell
instance (Foldable g) => listElts :: g a -> [EltIn g a] 
listElts as = zipWith EltIn (repeat as) (toList as)

listElts [1,2,3]
> [EltIn {_in = [1,2,3], _point = 1}, EltIn {_in = [1,2,3], _point =
> 2}, EltIn {_in = [1,2,3], _point = 3}] 
```

Now we are free to pass `EltIn g a` inhabitants to our functions to statically
enforce the element membership property (assuming `listElts` is the only way to
construct the inhabitants, of course). It's great that we can go so quickly from
a specific case for lists to the general case for arbitrary Foldables.
