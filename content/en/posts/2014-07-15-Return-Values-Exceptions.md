---
title: "Return values vs. Exceptions"
date: 2014-07-15
tags:
  - errors
  - exceptions
---

In imperative programming circles (particularly C++) there has been
[debate](http://stackoverflow.com/questions/1849490/c-arguments-for-exceptions-over-return-codes)
over whether it is better to use return values or exceptions to propagate
information about exceptional conditions. First of all there is the usual
distinction to be made between errors and exceptions: an **error** is an
unexpected condition arising from a mistake in the program code, while an
**exception** is an *expected* condition that requires special handling. This
post will be about exceptions, not errors, and I will strive not to mention such
conflations as "exceptional errors".

As noted elsewhere, return values have some advantages over exceptions, probably
the most important of which is that one maintains explicit control flow without
introducing myriad invisible exit points that can muddle with things like
resource acquisition/release (RAII is the standard response to this problem
with exceptions). The major flaw of return values is that they can be ignored in
C and C++: one can simply call a function that could return an exceptional value
and not bother to check the result. In practice this can result in silent
failures in production code where an intended side-effect has not been executed.
The second most common criticism is that return values do not allow one to
cleanly separate exception-handling code from "usual" control flow, but to me
this seems like a red herring: would the reader not want to know as soon as
possible after function invocation how the possible exceptions would be dealt
with?  Otherwise, there is an invisible link created between the site of
function invocation and the corresponding error-handling code, so again the
control flow becomes unclear. This is also unacceptable, no?

Part of this seems to be language limitations at play: in C++, one generally
does not think to *pass the exception handler itself* to the function/method
that could encounter the exceptional condition, for instance. In a functional
language this could be done by defining the custom handler in a `let` or `where`
binding and passing it as a first-class function.

Now consider the `Either E Int` type where `E` is an exception type. Inhabitants
of this type are constructed via either `Right :: Int -> Either E Int` or 
`Left :: E -> Either E Int`. In particular, any invocation of a function returning
this type **must deal with the possibility of exceptional conditions** (at least
if the result is to be used for anything).  Instead of the exception `E` being
an out-of-band value which alters control flow, it is reified in the type system
and **control flow proceeds as usual**. So in a single stroke we have obtained

1) predictable control flow; and 
2) forcing the caller to deal with any exceptional conditions. 

A function `either` can be defined which takes a handler
for `Left` values together with the handler for the usual program flow,
**thereby also making the link between the function invocation and the exception
handler explicit** (not invisible!) in the source code. One might contest that
this forces the caller to always determine whether the returned value is a `Left`
or `Right`, but 

1) modern CPUs have branch prediction that might be able to deal with this; and 
2) does it matter if the solution is this elegant?

