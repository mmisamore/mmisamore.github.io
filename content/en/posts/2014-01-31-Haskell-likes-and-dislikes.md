---
title: "Haskell: The Good, the Bad, and the Ugly"
date: 2014-01-31
tags: 
  - haskell
---

I am working on my first significant little project in Haskell; some code
should appear on Github in the not-too-distant future. This has spawned some
thoughts on the language as a whole:

## The Good Parts:

* **Pure functions**. This is probably the single biggest win versus most
mainstream languages. If you don't believe me, listen to John Carmack talk
about his experience: [youtube link](http://www.youtube.com/watch?v=1PhArSujR_A).  The ability to
statically guarantee that a function is free of side-effects is *absolutely
huge*, and opens up all sorts of opportunities for cleaner code.

* **Static type-checking**. Guarantee at compile-time that the inputs to a
function must satisfy specified properties. This can be done with smart
constructors, but also via static guarantees in the type system itself,
e.g. [Brent's blog](http://byorgey.wordpress.com/2010/06/29/typed-type-level-programming-in-haskell-part-i-functional-dependencies).

* **Immutable data, referential transparency**. Enables declarative-style
programming and equational reasoning about the meaning of code. Factoring
out functionality is trivial: pull out some pure code and you have a new
pure function. 

* Corollary: **Variables are immutable**: never again will I have to run a
loop in my head to keep track of *n* odd variables, *m* of which are updated
upon satisfaction of *k* conditions that are evaluated on each iteration! I
am probably not alone in having spent *hours* of my life in the past
tracking down bugs in imperative loops with mutable variables.

* **No loops**. In particular, *no large, messy loops* like one often sees in
imperative business code. Unlike opening an imperative loop where you can
just play around mutating variables willy-nilly, Haskell forces you to
*think first* about what you are doing: what set of objects is this
iterating over? Does the order of iteration matter? Should the objects be
evaluated in parallel? Exactly what is being accumulated, and why? 

* Almost **trivial to add parallelism** to code: running on multiple cores is
often just a matter of evaluating the elements of a list in parallel.
[That's parList](http://hackage.haskell.org/package/parallel-3.2.0.4/docs/Control-Parallel-Strategies.html).

* Kick-ass **profiling and debugging tools**, e.g. cost center profiling and 
[threadscope](http://www.haskell.org/haskellwiki/ThreadScope_Tour).
In particular, these allow us to easily evaluate performance gains 
due to parallelism. [QuickCheck](http://www.haskell.org/haskellwiki/Introduction_to_QuickCheck1)
provides automated fuzz-testing: just provide your definition of what
"arbitrary" instances of your types should look like, and it can try to
break your functions with random instances of these types, or check that
your functions satisfy given properties. All of this stuff is just
"standard" - you probably don't need valgrind.

* **Pervasive laziness**. This is just plain cool: Haskell uses a non-strict
evaluation strategy for expressions by default, meaning that it doesn't
evaluate anything that it doesn't have to. People complain that this can
result in space leaks, but this is largely mitigated by modern libraries
and profiling tools (especially heap profiling). I have found that trying
to force strictness can actually slow my programs down, meaning that
sometimes strict programs evaluate more than necessary - probably without
even realizing it!

* Access to **referentially transparent local mutable state** when you
absolutely need it: the state thread monad.

* **REPL interpreter** (ghci) for testing code as it is developed. Most modern
languages have finally (almost) caught up to Forth in this regard. Forth
still provides much finer granularity over the compilation process. Type
inference here is a big deal: write functions and let Haskell figure out
in which type you've landed, and then import the signature when you are
finished.

* **Newtypes**. Thin (i.e. nonexistent at runtime) wrappers to give existing
types new behaviors.

* **Type classes**. A static (i.e. safe) alternative to the "duck typing" of
dynamic languages.


## The "Bad" Parts

* Coming from the imperative world, I had to get used to the idea that I
**couldn't stick in "print" statements everywhere** to understand why certain
functions were not working as expected, especially in loops. Functions like
"scanl" and "iterate" are helpful in this regard. Since an interpreter
is available, these sorts of errors do not happen all that often. Factoring
out functions to test them separately can help.

* **Syntax and parser errors** took some getting used to. I was confused for
a good while about how the parser treats the "$" function, and how to use
"$" in combination with "." without getting long error messages from ghci.

* Applying the wrong number of arguments to functions can result in some
very intimidating error messages at times. **The type system tries really
hard to make sense of your stupid mistakes**, sometimes to the effect of
making things even more confusing.

* You will go in assuming that you are smarter than the compiler. Believe
me: you have never met a compiler like GHC. It is **smarter than you**, and the
best thing to do is to let it help you rather than trying to fight it.
Do not try to guess the types of your functions: at first, you will be
wrong at least 50% of the time. GHC/GHCi will help you figure things out
until you gain more confidence.

## The Ugly

* Cabal is still kind of a joke with regard to **managing package
dependencies**. Sticking with the latest of everything tends to help, but
there is still the problem that you cannot have the same version of the
same package installed twice with different dependencies, and this can
lead to problems.

* **Profiling seems to be disabled by default**. Fortunately this is
relatively easy to fix by enabling "library-profiling" in the .cabal/config
file. But it's one of those idiosyncrasies of the distribution that I wish
hadn't gotten in the way of more productive work.

* I don't know of any easy-to-use (i.e. available, documented) **packaging
solutions** to take Haskell binaries and send them out as Windows setup
programs or Mac "app" packages.

* The presence of [Haddock](http://www.haskell.org/haddock) for module
documentation has led many Haskell developers to think that this counts
as all the documentation they need to provide. The rest of us are then left
with **Haddock modules which are quite inscrutable and intimidating to 
newcomers**. This seems like a pretty serious problem: lots of libraries are
totally useless for lack of proper (i.e. non-Haddock) documentation in
addition to the usual module documentation.

