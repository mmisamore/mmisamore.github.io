---
title: Maybe [a]? 
date: 2015-07-13
tags: 
  - haskell
  - semantics
  - scala
---

I've been doing some coding in Scala just to get some experience with functional programming on the JVM, and part of
this has involved bringing in useful abstractions from Haskell as I needed them (scalaz notwithstanding).

Lately I ran into a pattern where the initial versions of some of my functions returned something of type `Maybe [a]`,
only to apply the mapping `Nothing -> []` and `Just xs -> xs` down the line. This is due to an interesting distinction
in semantics: "I ran into a problem and therefore there was no answer" (`Nothing`) versus "There was no answer". What can
one do with a `Maybe [a]` in practice if the term is `Nothing`? Well, we were expecting a list, and got nothing back, so
what does that mean for the rest of our computation if we want to remain total?

At this level of abstraction, it seems to me that the only thing we can do is to act as if we received `[]` instead
(since we have no idea what went wrong), but then we may as well have been using `[a]` to begin with! This becomes
especially clear when we inevitably attempt to `fmap` over a `Maybe [a]` to do something with the result: it is much more
uniform to just take a `[a]` as input and map over it, and if the input list happens to be empty then so be it. 

A better way of looking at this, however, is that `Maybe [a]` is the wrong abstraction: if I want to know if something
went wrong with retrieving a list, then it's probably a good thing to know why, especially when performing `IO`.
Ignoring the `IO` piece, this leads to `Either String [b]`, where `Either a` is a monad. In Scala I set this up as a
right-biased monad in the sense that failures of earlier computations trump later failures: this has the advantage of
identifying the sources of problems early and then bubbling them up the stack, much like unchecked exceptions in
imperative languages. But unlike unchecked exceptions, the type-system reminds me that the relevant functions can fail.

*Using this pattern, I cannot accidentally call a function without remembering to handle its exceptional return states.*

This leads to some nice fishy-ness too: if I start with a source type of `[a]`, I can compose

```haskell
f :: [a] -> Either String [b]
g :: [b] -> Either String [c]
```

to get `f >=> g :: [a] -> Either String [c]`. So I can have my cake and eat it too: perfectly composable computations
with informative failures in Scala, borrowing relatively trivial abstractions from Haskell.

Anyway, implementing even mildly complex abstractions like these with Scala turns out to be a pain in the ass: the lack
of global type inference means that return type inference fails exactly when you need it the most, and the type checker
doesn't seem smart enough to unify things like `a -> b` with `(c,d) -> e` (it seems to want to treat these as functions with
different arity - probably a holdover from Java thinking). The reason I'm currently sticking with Scala is that it
enables a sliding scale from the OO-world to the functional world, but maybe I should just be writing my little
libraries in Frege instead.

