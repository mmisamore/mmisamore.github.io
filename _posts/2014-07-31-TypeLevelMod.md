---
layout: post
title: Type-level modular arithmetic 
tags: [type-level]
---

So, a modern way to do the natural numbers at the type level is to take
advantage of the relatively recent "DataKinds" extension to GHC, which
automatically promotes (many) types to kinds and their type constructors to
constructors for these kinds.

    {-# LANGUAGE DataKinds, TypeFamilies, TypeOperators, GADTs #-}

So here they are:

    -- | Type for natural numbers
    data Nat = Z | S Nat 

*DataKinds* automatically turns this into a lifted kind 'Nat together with
constructors 'Z :: Nat, 'S :: Nat -> Nat (it is probably worth noting that these
lifted types are not inhabited). Now type-level addition is easy enough to
define:

    -- | Type-level natural number addition
    infixl 6 :+
    type family (n :: Nat) :+ (m :: Nat) :: Nat
    type instance 'Z :+ m = m
    type instance 'S n :+ m = 'S (n :+ m)

    ghci> :set -XTypeOperators
    ghci> :set -XDataKinds
    ghci> :kind! S Z :+ S Z
    S Z :+ S Z :: Nat
    = 'S ('S 'Z)

We can also do stuff like compare for equality or introduce type-level proofs of
ordering:

    -- Natural number order comparison
    infix 4 :<
    type family (n :: Nat) :< (m :: Nat) :: Bool
    type instance n :< Z = 'False
    type instance Z :< (S m) = 'True
    type instance (S n) :< (S m) = n :< m
  
Here is a type-level conditional operator for if-then-else:

    -- Type-level if-then-else
    type family ITE (b :: Bool) t e
    type instance ITE 'True t e = t
    type instance ITE 'False t e = e

    ghci> :kind! ITE (S Z :< Z) Int Char
    ITE (S Z :< Z) Int Char :: *
    = Char

The [standard first
step](http://www.reddit.com/r/haskell/comments/2airo1/with_datakinds_can_we_use_integer_as_typelevel)
in creating a type for modular arithmetic is to use a
phantom type parameter that keeps track of the modulus:

    newtype Mod (n :: Nat) = Mod Int
    instance Show (Mod n) where show (Mod m) = show m

Here Nat is a perfectly good *kind constraint* on the phantom parameter n, so
Mod is a type constructor of kind Nat -> *, and each such type Mod n has a data
constructor Mod :: Int -> Mod n. We want to be able to define addition as
something like

    plus :: Mod n -> Mod n -> Mod n
    plus (Mod m1) (Mod m2) = Mod $ (m1 + m2) `mod` (reify n)

where reify takes the type-level natural n to the corresponding term-level
natural. GHC [provides such
functionality](https://www.haskell.org/ghc/docs/latest/html/users_guide/type-level-literals.html)
via a "KnownNat" class in GHC.TypeLits. This is essentially a bit of magic
mapping the official type-level naturals to value-level naturals. 

Before this official solution appeared, various ad-hoc methods were used by
libraries, including [representing type-level naturals one digit at a
time](http://hackage.haskell.org/package/type-level-0.2.4/docs/src/Data-TypeLevel-Num-Sets.html).
To make the reification go, the latter library uses a type constructor :* of
type * -> * -> *, and one can strip off trailing digits at the type level by
pattern matching:

    div10Dec :: x :* d -> x
    div10Dec _ = undefined

Once you can pattern match on digits then you are good to go since a typeclass
for reify need only be defined in terms of decimal digits. The analogous
function for our type Nat unfortunately fails to typecheck due to
technicalities:

    > let subS :: S n -> n; subS _ = undefined
    Kind mis-match
    Expected kind `OpenKind', but `S n' has kind `Nat'
    In the type signature for `subS': subS :: S n -> n

The analogous definition

    let subS :: a :* b -> a; subS _ = undefined

compiles without a hitch, so we are definitely dealing with limitations of the
typesystem here. Anyway, that's enough of the deep dive into the GHC type system
for now. 

