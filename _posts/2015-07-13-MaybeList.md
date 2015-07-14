---
layout: post
title: Maybe [a]? 
tags: [haskell, semantics]
---

Lately I ran into a pattern where the initial versions of some of my functions
returned something of type Maybe [a], only to apply the mapping Nothing -> []
and Just xs -> xs down the line. This is due to an interesting distinction in
semantics: "I ran into a problem and therefore there was no answer" (Nothing)
versus "There was no answer". What can one do with a Maybe [a] in practice if
the term is Nothing? Well, we were expecting a list, and got nothing back, so
what does that mean for the rest of our computation if we want to remain total?

At this level of abstraction, it seems to me that the only thing we can do is to
act as if we received [] instead, but then we may as well have been using [a] to
begin with! This becomes especially clear when we inevitably attempt to fmap
over a Maybe [a] to do something with the result: it is much more uniform to
just take a [a] as input and map over it, and if the input list happens to be
empty then so be it. 

This pattern happens to generalize to monoidal computations: if we have some
computations of the form f_i: x -> Maybe a where "a" is a monoid whose results
we intend to fold over anyway, then it seems there is no harm in replacing these
with g_i: x -> a where the values that were previously "Nothing" now map to the
neutral element, which are not distinguished in practice from it. This
uniformity is quite pleasing. 

