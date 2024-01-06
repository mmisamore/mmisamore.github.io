---
title: Fun with Monogenic Semigroups 
date: 2014-09-17
tags: 
  - semigroups
  - automata
---

Finite semigroups are important in computer science in particular because they
can represent composition of state transitions of deterministic automata,
including non-invertible transitions. From an algebraic perspective they are
interesting because they can be very un-grouplike: for example, many interesting
examples of semigroups have non-trivial idempotent elements, so that \\(x^2 =
x\\) is possible without \\(x\\) being an identity element. The `Maybe` monoid is
an obvious first example of this.

Of course, every group and monoid (finite or not) has at least one idempotent,
namely \\(1\\), but in fact more generally every finite *semigroup* also has an
idempotent. To show this, it suffices to take any element \\(x \in S\\) of a
finite semigroup \\(S\\) and consider the *finite monogenic semigroup* \\( < x > \\)
that it generates. If we can show that \\(< x >\\) contains an idempotent then we
are done.

**Lemma**: Every finite monogenic semigroup contains at least one idempotent.

**Proposition**: Every finite non-empty semigroup \\(S\\) contains at least one
idempotent. 

*Proof of the Proposition*: consider any element \\(x \in S\\) and observe by the
Lemma that \\( < x > \\) contains an idempotent, hence \\(S\\) contains an
idempotent. QED.

*Proof of the Lemma*: suppose the monogenic semigroup is \\( < x > \\) for some
generator \\(x\\), and let \\(k\\) be the smallest exponent such that \\(x^k =
x^l\\) for some \\(l < k\\). Now if \\(2l < k\\) then we can argue that

\\(\qquad x^{k-l} \cdot x^{k-l} = x^{k-l} \cdot x^l \cdot x^{k-2l} = x^k \cdot x^{k-2l} = x^{k-l}\\)

since \\(x^k = x^l\\), and one can observe that in the group case \\(x^{k-l}\\)
is just the identity element. The ability to form the element \\(x^{k-2l}\\) is
crucial here.

Otherwise, \\(2l \geq k\\), but then \\(x^{2l} = x^p\\) for some \\(p < k\\)
since any power of \\(k\\) can be replaced by a power of \\(l < k\\) instead.
Then if \\(x^k = x^l\\), then \\(2k\\) should act like \\(2l\\) which acts like
\\(p\\), so \\(x^{2k-p}\\) is a candidate idempotent. Furthermore \\(2p < 2k\\)
so that \\(x^{2k-2p}\\) is a legitimate element of the semigroup. Now observe

\\(\qquad x^{2k-p} \cdot x^{2k-p} = x^{2k-p} \cdot x^p \cdot x^{2k-2p} = x^{2l}
\cdot x^{2k-2p} = x^p \cdot x^{2k-2p} = x^{2k-p}\\)

so again we have an idempotent. QED.

I liked coming up with these arguments since they forced me to go outside my
comfort zone of thinking in terms of groups, especially with respect to having
negative exponents available. It's also kind of surprising that a statement like
this could be true in this generality. Of course, there are *infinite*
semigroups that have no idempotents at all (and therefore no identity element):
the canonical example is \\((\mathbb{N}_{>0}, +\\)), so finite semigroups are pretty
special in this way. There are also infinite semigroups where *every* element is
an idempotent: my favorite example of this so far is \\((\mathbb{Z},
\mathrm{min})\\).

