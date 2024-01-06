---
title: Deriving Type Derivatives 
date: 2015-04-19
tags: 
  - haskell
  - derivatives
---

The polymorphic list type `[a]` can be modeled as 

```haskell
List a = Empty | Cons a (List a)
```

which looks an awfully lot like \\(L = 1 + X\cdot L\\). The corresponding
geometric series yields a closed form expresssion \\(L = \frac{1}{1-X}\\) and
the derivative with respect to \\(X\\) is

$$L' = -\frac{1}{(1-X)^2}\cdot(-1) = \frac{1}{(1-X)^2} = L^2$$

Thus the derivative of \\(L\\) can be modeled as a list of all elements before
the hole together with another list of all elements after the hole. This is a
nice result, but I was thinking about it another way: starting with the equation
\\(L = 1 + X\cdot L\\) we can certainly *implicitly* differentiate to get
    
$$L' = 1\cdot L + X\cdot L'.$$

This is just a recursive way of saying that the deriviative is just a list of
remaining elements since the hole was at the beginning, or it is an element
\\(X\\) followed by a list with a hole in it. Expanding this and treating
\\(X'\\) like \\(1\\) yields something like

$$L' = L + X(L + X\cdot L') = L + X\cdot L + X^2 \cdot L + X^3 \cdot L + \ldots$$

which was actually the representation I had initially thought of when trying to
differentiate the list type. The problem with this type of representation, of
course, is that it's an infinite sum type which, in general, seems to require a
countably infinite number of constructors (in Haskell).

More importantly, however, we have lost some information when taking these
deriviatives: the information about where the hole is located. Sure, it is
implicitly there (most helpfully in the pair-of-lists form), but it seems to me
that if we are taking a derivative of something like \\(X^2\\) then it would be
more useful to have something of the form

$$H\cdot X + X \cdot H$$

where the hole is represented by \\(H\\). Returning to the above recursive
expression for the deriviative, we would have instead

$$L' = H\cdot L + X \cdot L'$$

which is what we said in prose anyway! This new form is useful because it
combines the advantages of a) not requiring infinitely many constructors; and b)
explicitly telling us where the hole is. Writing this down as a Haskell type and
firing up GHCI, we arrive at 

```haskell
data L' a = HoleFirst [a] | HoleLater a (L' a)
```

Taking a cue from Conal Elliot's writings on derivatives, I want to build a type
class whose inhabitants are differentiable types together with their derivatives
and functions to fill the holes.

```haskell
-- Types with derivatives and hole-filler functions with args of type a
class DerivType t a where 
  type Deriv t :: *
  fill :: Deriv t -> a -> t

instance DerivType [a] a where 
  type Deriv [a] = L' a 
  fill (HoleFirst (as :: [a]) a = a : as 
  fill (HoleLater x as) a = x : (fill as a)
```

This works, but leads to type inference trouble without explicit type annotation
everywhere:

```haskell
> (fill (HoleFirst [1,2,3::Int]) (0::Int)) :: [Int]
[0,1,2,3]
```

The trouble seems to be that the compiler knows that the first argument of
`fill` is some `Deriv t`, and (I think) it knows that `Deriv t ~ Deriv [a] ~ L' a`,
but it cannot deduce that `t ~ [a]`. This is reasonable since Deriv is not
monomorphic. What about an approach via type families? This seems to work out a
lot better overall:

```haskell
type family DerivPair t
type instance DerivPair [a]   = (L' a)
type instance DerivPair (a,b) = (DerivPair a, DerivPair b)

type family Elem t
type instance Elem [a]   = a
type instance Elem (s,t) = (Elem s, Elem t)

class (Elem t ~ a, DerivPair t ~ dt) => Derivative dt a t where
   fill :: dt -> a -> t

instance Derivative (L' a) a [a] where
   fill (HoleFirst as) a = a : as
   fill (HoleLater x as) a = x : fill as a

instance (Derivative ds a s, Derivative dt b t) => Derivative (ds,dt) (a,b) (s,t) where 
   fill (ds,dt) (a,b) = (fill ds a, fill dt b)                                                                          
```

We are able to provide the compiler with info about the element types of the
collection and the relationship between `t` and `dt`. This also demonstrates how to
deal with composite derivatives.

```haskell
> fill (HoleFirst [1,2,3]) 0 :: [Int]
[0,1,2,3]

> fill (HoleLater 0 (HoleFirst [2,3])) 1 :: [Int]
[0,1,2,3]
```

GHCI just wants to know the return type we are expecting, which seems
reasonable enough. This framework is also extensible:

```haskell
> let h1 = HoleFirst ["a","b","c"] :: L' String
> let h2 = HoleLater 0 (HoleFirst [2,3,4]) :: L' Int
> fill (h1,h2) ("x",6) :: ([String],[Int])
(["x","a","b","c"],[0,6,2,3,4])
```

Great! We have a way not only of taking type derivatives, but also of keeping
track of the locations of the holes so they can be filled in later. We also have
enforcement of the relationship between each type and its derivative in the type
system, and the type system guarantees that we can only fill in the holes with
the proper element types. A totally separate approach to this stuff is via
lenses: rather than poking an actual hole in a type we can just return a lens
onto a particular element and provide a default value.

What if we wanted to differentiate `L' a` to reach all two-hole lists? Well, we
have a type equation

```haskell
L' a = [a] :+: (a :*: L' a)
type a :+: b = Either a b
type a :*: b = (a,b)
```

so it seems to me that we can encode the standard rules for differentiation
following Conal Elliot, but I'll leave that for another day.

