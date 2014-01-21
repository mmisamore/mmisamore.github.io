---
layout: post
title: Fun with finite strings
tags: [strings, monoids]
---

I was recently inspired by [Lipton's blog](http://rjlipton.wordpress.com) to
have a look at a fun little problem about finite strings.

Fixing an alphabet Σ, a *finite string* *x* over Σ is just a finite
sequence of elements of Σ, and the *length* |*x*| of *x* is the number
of elements appearing in this sequence. Given two strings *x* and *y*
over Σ, there is a binary operation called *concatenation* which sends
the pair *x*, *y* to the string *xy* defined as the sequence of elements
of *x* over Σ followed by the sequence of elements of *y*. For example,
if *x* = "Hello", and *y* = " world!" then *xy* = "Hello world!".

The operation of concatenation makes the set of finite strings over Σ
into a *semigroup*, which just means that string concatenation is
associative. It is a *cancellative semigroup* in the sense that any
equation of the form *xy = zy* implies *x = z* (right cancellation), and
*yx = yz* also implies *x = z* (left cancellation). This semigroup is
*infinite* in the sense that it has infinitely many elements, since
finite strings can be arbitrarily long. If one includes the empty string
1 = "" as a finite string, then this semigroup is a *monoid* with
concatenation with the empty string defined by x1 = x, 1x = x for any
x.

Exercise for the reader: any *finite* cancellative monoid is a *group*
in the sense that any element *x* will have a unique *inverse* *x^−1^*
in the sense that *xx^−1^ = x^−1^x = 1*.

In his blog, Lipton mentioned the following theorem about the monoid of
finite strings over any alphabet Σ: given finite nonempty strings *x*
and *y*, if there exist positive integers *m* and *n* such that *x^m^* =
*y^n^* then *xy = yx*, and conversely.

Like Lipton, I found this result pretty surprising when I first saw it;
one way of reading it is that it gives a non-obvious necessary and
sufficient condition for two finite strings to commute! 

So here is my proof of this theorem: first, suppose *x^m^ = y^n^* for
some positive *m* and *n*. Then if |*x*| = |*y*| then by reading the
first |*x*| elements from each expression one sees that *x = y* so that
*x* and *y* must commute. So suppose *x ≠ y* and without loss of
generality suppose that |*x*| < |*y*|. The idea will be to get an
inductive argument going on the length of *y*.

There are at least two ways of reading the equation *x^m^* = *y^n^*: it
means on the one hand that *y = x^k^p~x~* for some nonnegative integer
*k* and some *proper prefix* *p~x~* of *x*, and it also means that *y =
s~x~x^l^* for some nonnegative integer *l* and some *proper suffix*
*s~x~* of *x*. This leads to the equations

|*y*| = *k*|*x*| + |*p~x~*| and
|*y*| = *l*|*x*| + |*s~x~*|

so that

(*k - l*)|*x*| + |*p~x~*| = |*s~x~*|

but then |*p~x~*| = |*s~x~*| since these nonnegative quantities are both
strictly less than |*x*|. But then (*k - l*)|*x*| = 0 so *k = l*, thus
*y = x^k^p~x~ = s~x~x^k^*. But then since |*p~x~*|, |*s~x~*| < |*x*| it
follows that *p~x~* is also a proper *suffix* of *x* and similarly
*s~x~* is a proper *prefix* of *x*, and |*p~x~*| = |*s~x~*| so *p~x~* =
*s~x~*. Therefore one can write *y = x^k^p~x~ = p~x~x^k^*, so we have
constrained the form that *y* could possibly take.

Now return to the original equation *y^n^ = x^m^*. Since *x^k^* commutes
with *p~x~* as established above (and *x^k^* commutes with itself), one
can rearrange to get

*y^n^* = (*x^k^p~x~*)...(*x^k^p~x~*)
= *x^kn^p~x~^n^* = *x^m^*.

For reasons of string length, it is impossible that *kn ≥ m* since
otherwise *x^kn−m^p~x~^n^* = 1 so that n = 0, hence m = 0, in
contradiction to the assumption that *m*, *n* > 0. Thus *kn < m* and
by cancellation *p~x~^n^* = *x^m−kn^*. Now we have an equation relating
powers of *p~x~* and *x* where the former is a proper prefix of the
latter, so is shorter, and |*x*| < |*y*| by assumption, so by the
inductive hypothesis this implies that *p~x~* commutes with *x* and thus
*x* commutes with *y = p~x~x^k^*. It remains to check the base case
where |*x*| = 1 and |*y*| = 2, but this is trivial.

The converse argument is similar but shorter: suppose *xy = yx* and
observe that if |*x*| = |*y*| then *x = y* so we are done in that case.
Otherwise we may suppose without loss of generality that |*x*| < |*y*|
and the commutativity condition then says that *x* is both a proper
prefix and proper suffix of *y*, so one may write *y = xs~y~* and *y =
p~y~x* for suitable strings *s~y~* and *p~y~*. Writing *xy = yx* and
using cancellation, one finds that *xs~y~ = s~y~x* and similarly for
*p~y~*. This implies that *s~y~* is both a *prefix* and a suffix of *y*
and similarly for *p~y~*, and since |*p~y~*| = |*s~y~*| one has that
*p~y~ = s~y~*. Thus *y = p~y~x = xp~y~*, so in particular *p~y~*
commutes with *x*.

Since |p~y~|, |*x*| < |*y*|, one may use an induction hypothesis on the
length of *y* to assume that there are positive integers *m* and *n*
such that *x^m^ = p~y~^n^*. But then *y^n^ = (p~y~x)^n^ = p~y~^n^x^n^ =
x^m^x^n^ = x^m+n^* so the result follows. It remains to check the base
case |*x*| = 1, |*y*| = 2, but again this is trivial. QED
