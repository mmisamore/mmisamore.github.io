---
title: Dependency Hell with Cabal
date: 2014-01-24
tags:
  - haskell
---

Today I got my first true taste of dependency hell in Haskell, via cabal
install (as expected). Actually it is rather easy to land in hell by attempting
to cabal install anything outside of the batteries-included Haskell Platform,
which is inevitable given that we all eventually want to use the latest of
this-or-that package. 

My particular issue was kind of interesting: I wanted to use packages A, B, and
C in the same source file, but B expected version A1 of A and C wanted version
A2 of A. The particular object that they both wanted was a data type D, which
had the same name in both A1 and A2. The language does not seem to permit
invoking D twice (once from A1 and again from A2), so the needs of B and C
could not be appeased simultaneously. Fortunately package B turned out to have
a newer version that depended on A1 (the newest version) instead, thus dodging
the problem in this instance. This is an example of 
a problem pointed out [here](http://www.haskell.org/haskellwiki/Cabal/Survival).

There has apparently already been a lot of talk about dependency hell with
Cabal. [Some](http://cdsmith.wordpress.com/2011/01/16/haskells-own-dll-hell)
have identified upper bounds on versions as part of the problem, but there is
also [recognition](http://cdsmith.wordpress.com/2011/01/17/the-butterfly-effect-in-cabal)
that more fundamental forces are at play. Fundamentally, in cabal **you cannot
build the same package twice with different sets of dependencies**. This smells 
like broken behavior to me.

Possible solutions include [using
snapshots](http://www.yesodweb.com/blog/2012/11/solving-cabal-hell) of something
more-or-less like the Haskell Platform. This is appealing in the sense that a
given package could be determined to be "known compatible" with various versions
of a standard platform, and unappealing in the sense that much of the "latest
and greatest" will not make it into a public version of such a platform anytime
soon (without resources that probably do not exist), significantly hindering new
development. This seems like a good idea only for local Hackage installs or
similar.

A lot of people have argued for relaxing or dropping upper bounds on versions,
but this practically guarantees future breakage somewhere due to API changes.  I
know that one argument says that there should be relaxed/nonexistent upper
bounds for reasons of propagating security updates from dependencies up to their
ancestors. To me this throws out the baby with the bathwater: one loses any
guarantee that the dependency is "known good" to the ancestor, so the software
may not compile/work at all after the dependency upgrade! (in which case
security is irrelevant anyway) 

No, I would rather depend on packages that can tell me exactly which versions of
which libraries they are known to successfully compile against, and those
dependencies should have known good dependencies as well, etc. Such a "strict"
approach to dependency management means that packages must guarantee that they
*will not break* upon a permissible version bump (specified by the package!)
of some dependency. In the correct sort of system, a computer should be able
to check for "known good" dependencies automatically - no guesswork involved.
If two packages depend on different versions of the same dependency, those two
versions should be installed side-by-side. Furthermore, in a future world these
sets of "known good" dependencies per package could be published and shared
online, reducing the workload to determine the known-good versions.

[This](http://nixos.org/nix) is the best I've found so far regarding how to fix
such issues in a sensible and reliable way. "\[...i\]f a package builds correctly
on your system, this is because you specified the dependency explicitly." Yay!
 
