---
title: Announcing directed-cubical 0.1.0.0 
date: 2014-02-19
tags: 
  - haskell
  - homotopy
  - quickcheck
  - threadscope
  - topology
---

I've just released a [new Haskell library](https://github.com/mmisamore/directed-cubical) 
on GitHub which I intend to get onto Hackage real soon now. It's called
"directed-cubical", and it contains a couple of modules for creating,
transforming, and reducing finite directed cubical complexes. The reduction
algorithms are based on my forthcoming paper "Computing Path Categories of
Finite Directed Cubical Complexes", which provides the theoretical justification
for why removing corner vertices determines a fully faithful functor on the
level of path categories. The idea is that the output complexes can be fed
into a software implementation of Jardine's algorithm for determining the
morphism sets of the path categories of the associated triangulations.

The module "DirCubeCmplx" had to be written first because I wasn't aware of any
pre-existing Haskell libraries for dealing with finite directed cubical
complexes. I ended up going with an 
[unordered-containers](http://hackage.haskell.org/package/unordered-containers-0.2.3.3)
approach which essentially uses functional hash tables, and the performance
turned out to be pretty acceptable. There's a certain amount of memoization
built-in as well since a lot of cubical substructures can be computed once for
the "generic" case and then "translated" into place for the non-generic
situation. It's possible that storing the top-cells in a more structured type
could yield improvements to the algorithms that rely on "cell neighborhoods",
maybe via some sort of BSP technique.

The other module, "CornerReduce", contains algorithms that search for so-called
"corner vertices" and remove them, calculating the new top-cells and corner
vertices that may result. Iterating this process yields "corner-free" complexes,
so this functions as a sort of "thinning" algorithm for directed spaces
rather than ordinary topological spaces. I put a certain amount of effort into
determining cases where corners could be removed simultaneously, and this
resulted in a modest speedup over a pure serial algorithm.

One of the more fun aspects of this project is that the problem is
essentially massively parallel: by using a partitioning technique and help
from [threadscope](http://www.haskell.org/haskellwiki/ThreadScope), I was able
to get 7.4x speedup on my 8-core desktop machine. Adding multicore parallelism
turned out to be relatively easy: it was just a matter of parListing and
parBuffering in some appropriate places, and the RTS stats made it really 
obvious when I was wasting sparks. 

[QuickCheck](http://hackage.haskell.org/package/QuickCheck) was pretty amazing
at finding problems with my attempts at suitable "Arbitrary" instances, and I
was also able to use "unGen" from that library to create a simple "rList"
function that was suitable for fuzz-testing the functions.

This is my first substantial project in Haskell, and I have to say that I'm
pleased with the language so far. Being able to write pure, reusable, composable 
functions is a wonderful thing. [Unrolling loops with
"iterate"](/posts/2014-02-15-iterate/) decouples them
from their analysis, so even the loop analysis tools become reusable! I suppose
this could be done in an imperative language using coroutines, but I've never
seen this done in practice; **usually the imperative gut instinct is to drop
side-effects into the loop itself to discover the internal state** (which is
the road to perdition as far as I'm concerned). Considering how many gigabytes
of data the functions generate and process, it's also a bit shocking (and
pleasing) to see only tens of megabytes actually consumed even in large-ish
examples - I suspect this has quite a bit to do with the non-strict evaluation.


