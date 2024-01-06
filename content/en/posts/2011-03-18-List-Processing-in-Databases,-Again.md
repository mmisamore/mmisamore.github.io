---
title: List Processing in Databases, again...
date: 2011-03-18
tags: 
 - sql
 - list processing
---

List processing and functional programming style have now come up more
than once as a solution to my database programming woes, although by
"list processing" I often just mean "set processing" (ordering only
matters for the final resultset). I have found that the combination of
the following two principles is very powerful for any type of software
development:

1. A function should do one thing, and one thing only; and
2. It should be easy to compose functions.

If you are coding in C or C++ or Python, you are probably *not* in the
habit of doing #1, despite paying the occasional lip service to the
idea. My impression is that imperative language programmers do not
usually take the "one thing" part seriously enough. But I cannot really
blame them for this since I have also found that it is difficult to make
a language like C or Python do #1. This is in part because

1. Typical style is to define a boatload of mutable local variables;
2. The overhead of writing new functions is typically higher than
writing in Forth or in a functional language, partly because of having
to pass all of these variables around;
3. It is "too easy" to write a "function" that does everything in one
shot;
4. Without features like parametric polymorphism, currying, Functors,
etc. it is difficult to reuse functions anyway, at least in strongly
typed languages.

I would argue that *most of the time* principle #1 is broken by
people doing imperative coding. Object-oriented programming and subtype
polymorphism mitigate this problem somewhat, but overall I find the
resulting solutions too complex to be satisfactory. Python uses dynamic
typing instead, but in the process commits the sin of consuming the
user's runtime resources to aid the programmer (with the runtime type
system) when static strongly typed languages like Haskell have shown
this to be unnecessary.

As for principle #2, it should be obvious that it becomes easier to
compose functions when there are only 1 or 2 parameters (3 tops), and
with good factoring that should really be the upper limit on their
number. Forth will force you to understand this principle and live with
it, and Haskell probably will too.

It is *incredibly powerful* to be able to compose functions! Probably
my favorite example of function composition is for list-valued functions
taking lists as parameters. This is ideal for information processing in
databases where you "get a list of such and such" and continually
transform it into new, more complex, lists. In 2002-2003, I discovered
this approach when using Tcl for database programming, and in 2011 I am
returning to it again, this time in SQL.

List processing has been lurking in the background in SQL (by which I
mean TSQL for the purposes of this entry) for quite some time already: a
"select" statement coded as a Common Table Expression is no more and no
less than a virtual table or "list"! A typical SQL query programming
style is to cascade a number of CTE definitions and then to perform a
final select statement that binds together and extracts all of the
relevant fields from the CTEs, and of course some CTEs depend upon the
results of others so that one really is doing list processing at some
level.

CTEs are not reusable, however, except insofar as one can cut and paste
their source code. For reusable list processing in TSQL for querying
purposes, there appears to be one "best" solution: user-defined
functions taking table types as parameters that return tables. To give
an idea of how backwards SQL programming is (at least Microsoft's
version of it), this functionality was only introduced in 2008. But
anyway, this does appear to be a good solution for function composition,
and I cannot wait to try it out. I hope to be able to write statements
like:

```sql
select col1, col2
from function1(function2(table1), function3(table2)) f
     join function4(table1) g on g.blah1 = f.blah2
```

Quite terse, and quite powerful. And the best part is that the functions
are stored in the database so they can be used by everyone; there is a
lot of potential there for building a standard library of user-defined
functions. Should be interesting to see what happens.
