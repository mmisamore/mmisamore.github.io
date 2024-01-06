---
title: Extensible effects and understanding functional patterns
date: 2014-06-12
tags: 
  - haskell
  - effects
---

I recently read the [paper](http://www.cs.indiana.edu/~sabry/papers/exteff.pdf)
on extensible effects in Haskell. It was good to see open union types being used
for something like this, and the coroutine-based handlers seemed easy enough to
write. [Edward Kmett](https://github.com/ekmett) pointed out that it essentially
looked like a CPSed free monad on a coproduct, much like a [datatypes a la
carte](http://www.cs.ru.nl/~W.Swierstra/Publications/DataTypesALaCarte.pdf)
approach. Going over to read the original `a la carte paper explained some of
the other blog posts and comments I had read elsewhere.

More significantly, I am actually starting to understand Ed Kmett's writings
(like the above), which must mean I've learned something! I think it was a
reddit comment of his that finally triggered my understanding of how the Yoneda
embedding yields a Functor instance that allows one to combine two fmaps into
one without retraversing the structure. I'm not sure if there is an easy way to
do this in an imperative language.

Yoneda is one of those "functional patterns", like monads, applicative functors,
or monoids, which seem like abstract nonsense until you understand that they
have a very practical purpose. **They allow you to get stuff done**, often more
efficiently/easily than in other mainstream languages (and Haskell is [now at
Facebook](https://code.facebook.com/projects/854888367872565/haxl), so it's
probably safe to say that it is going mainstream). These functional patterns are
just types which satisfy special properties (expressed via typeclass constraints
and equational laws). **These patterns give code that is more structured than
classical "structured programming"!** Iterative loops, for example, can have any
number of possible uses, and the compiler is left guessing what the programmer
actually wants to do. When your loops are coded as applications of map or
iterate, the desired semantics are suddenly *much* clearer to both humans and
compilers. 

Kmett's [blog](http://comonad.com) posts on
[Codensity](http://comonad.com/reader/2011/free-monads-for-less) explain pretty
clearly why Codensity shows up as a restricted form of the ContT
monad-transformer, and makes it at least plausible that Codensity gives both the
CPS-style speedup for free monads together with an efficient Functor instance
like the Yoneda embedding. Sweet!


