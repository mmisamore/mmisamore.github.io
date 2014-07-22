---
layout: post
title: Traversals as Natural Transformations 
tags: [traversals, functors, natural transformations]
---

Generally speaking, data types "F a" which act as containers are *functors* in
the sense that given a function \\(f: a \to b\\) one can produce a corresponding
function \\(F(f): F(a) \to F(b)\\) via the "fmap" for F. If the kind F is also
traversable then there must be a function \\(t: F(a) \to [a]\\) and similarly
for "F b".

Now morally, if one has any consistently defined traversal for F it should go
through the elements of "F a" while ignoring the actual "a" values, and
similarly for "F b". By "consistently defined" I mean that traversing "F a"
followed by applying \\(fmap\, f\\) to the resulting list of values should be
the same as applying \\(F(f)\\) first to obtain a structure of type "F b" and
then traversing to obtain the resulting list of type "[b]", regardless of how
"a" and "b" are defined.

We have just described a natural transformation from F to []! For example, any
inorder traversal of a tree yields a list of the values in the tree based on the
structure of the tree itself (which depends completely on the ordering of the
values), and if \\(f: a \to b\\) is any computable homomorphism of totally
ordered types then this yields a natural transformation 
\\(inorder:\, Tree \to []\\). 

Pre- and post-ordering traversals seem a bit more interesting because they
depend on the actual structure of the tree rather than the underlying totally
ordered set: it is easy to create trees that have the same in-order traversal
but differing pre-order traversals, for example. Nevertheless, if the underlying
structure of the tree is not altered by the function \\(Tree(f)\\) for any
order-preserving homomorphism \\(f\\) (it is somehow "stable", in other words)
then the pre-order traversal is also a natural transformation in this sense.

It seems like this notion should extend to essentially all container types with
respect to the type constraints on their inputs insofar as there is a
deterministic way to list their elements.



