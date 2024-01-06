---
title: Continuations and Free Algebras 
date: 2016-03-12
tags: 
  - monoids
  - continuations
  - category theory
  - haskell
---

The comonad reader has a [lovely
article](http://comonad.com/reader/2015/free-monoids-in-haskell/) about the
proper definition of free monoids in the category of Haskell types. By thinking
about universal properties in category theory, it is easy to see that the proper
type for \\(f a\\), the free monoid on any type "a", is given by

$$f a = \forall m. \textrm{Monoid } m \Rightarrow (a \to m) \to m.$$

This definition is obviously implicit: the free monoid on "a" is the universal
thing such that when given a map \\(a \to m\\) where "m" is any monoid, we can
map arbitrary elements of that thing into "m". Supposing we have elements \\(f,
g\\) of the free monoid \\(f a\\), their product is just the pointwise product
\\(\lambda: h \to f h \cdot g h\\) in \\(m\\). The unit map \\(\eta: a \to f a\\)
of the corresponding adjunction is the only thing it could be: evaluation with
flipped arguments, and similarly for the map giving the universal property of
the adjunction.

But there is no reason we need to stick to monoids: the upshot is that this
extends to all types of algebraic constraints on \\(m\\) such as [free modules over
semirings](http://comonad.com/reader/2011/free-modules-and-functional-linear-functionals/).
For sets, we usually think of a "free algebra" for some signature as the thing
that we get by starting with the underlying elements and combining them with the
given operations to formally generate new elements. This usually results in a
much larger underlying set than we started with. We can imagine weakening the
constraints on \\(m\\) from monoids to semigroups to magmas and then... nothing.
But what do we get in this limit? Here is the type:

$$\forall r. (a \to r) \to r.$$

This is the "universal" thing one gets on a type "a" by taking the closure under
no operations whatsoever. Without the quantifier on "r" this would be the
continuation type "Cont r a", and with the quantifier it turns out to be the
condensity of "a" with respect to the Identity functor: "Codensity Identity a".
The unit map \\(\eta\\) of the adjunction still exists (as flipped evaluation),
and feeding the identity map on "a" to the continuation collapses it back down
to "a". The fact that the continuation must work for *every* "r" makes this
distinct from a double-negation in particular, so we have "fattened up" the type
"a", but by so little that we have an isomorphic type. In fact, this
construction is a natural isomorphism by the Yoneda lemma: replace the target
"r" with "Identity r".

Generalizing a bit to the full codensity along an arbitrary functor "f" gives
the type

$$\textrm{Codensity}\\,f\\,a = \forall r. (a \to f r) \to f r$$

In particular there is still a map \\(\eta: a \to \textrm{Codensity}\\, f\\, a\\)
given by flipped evaluation at "a", but we have fattened up "a" via "f" so we
cannot expect to be isomorphic to "a" anymore. Nevertheless, if "f" admits a
"pure" function ala Applicative then we can construct "pure a" which yields an
element of "f a" after applying the continuation. If "f" is a Monad then we can
go the other direction:

$$\textrm{lift} :: f a \to \forall r. ((a \to f r) \to f r)$$

is essentially just ">>=" for f. In any event, there is also an interesting map

$$\textrm{fmap}\\, \eta: \textrm{Codensity}\\, f\\, a \to \textrm{Codensity}\\, f\\, (\textrm{Codensity}\\, f\\, a)$$

This means that within any Codensity computation I have access to the
continuation itself; this reminds me of call/cc for ordinary continuations.
