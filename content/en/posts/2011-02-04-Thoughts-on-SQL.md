---
title: Thoughts on SQL
date: 2011-02-04
tags: 
 - sql
---

My current job involves coding a lot of SQL queries and has had me
thinking about relational database concepts in general. I have come to
believe that SQL queries of 100 lines or less are "simple", that queries
of about 500 lines are moderately complex, and that queries of over 1000
lines indicate that one is starting to really use the language seriously
(provided most of those lines are not simply due to redundancy). The
"advanced" features such as "partition by", user-defined functions,
dynamic SQL, etc. are really quite easy to learn if one has any serious
programming background. I do not mean to say that they are *all*
pleasant to use; far from it.

Using stored procedures, user-defined functions, and control structures
in SQL is reminiscent of BASIC programming in the 1980s; there is
certainly not the richness and flexibility of modern programming
languages here. It will probably upset some people that I refuse to use
uppercase syntax in SQL, but I prefer not to look at "SELECT", "FROM",
"WHERE", etc., and it is easier to type in lowercase despite having a
caps lock. As far as I can tell features like polymorphism are not even
on the radar with SQL, but at least the presence of ''exec()'' gives one
a limited low-level opportunity to emulate them. Variables are typed
(and the types must be declared, which is a plus to me), and the
typecasting happens automatically.

Programmers used to imperative languages would probably hope to be able
to do a huge table join, import all of the relevant fields into
variables, and then do a bunch of ad-hoc processing of these variables
to produce the desired result. SQL forces you to be more structured than
this, and this aspect of the language is probably desirable from a
quality-assurance perspective. One often wants to see and examine lots
of intermediate result-sets to verify data quality and code correctness,
and this may or may not be convenient to do from imperative code; it
*is* convenient to do from any modern SQL IDE.

From my experience on internet forums, there are a lot of *bad* SQL
programmers out there; I am not surprised that SQL injection
vulnerabilities remain a problem. On the other hand, much of the syntax
of SQL is unintuitive and the vendor documentation tends to suck, so it
is often more helpful to see how things are done "in the field". This
requires that one can distinguish the good ideas from the bad ones, but
is helpful nevertheless.

Common Table Expressions are one of the best features of SQL, and I use
them extensively. When I was first learning I would often use temporary
tables or table variables to store intermediate result-sets, but have
since moved away from them since they both require more maintenance than
a simple

```sql
;with CTE as (blah) select * from CTE
```

where the code above the "select" can be copied into another query for
testing and one does not have to worry about dropping tables or
declaring table variables. I often use cascades of CTEs to build up
queries from modest starting result-sets, adding more information with
each new query. This approach is effectively "factoring" in SQL, and I
have found with experience that modifying existing code became much
easier with this approach, as the change usually only had to appear once
in a CTE, and that CTE was inevitably a relatively simple expression to
understand. I suppose in that sense I take the "Forth" approach to SQL!

One thing that I like about relational databases is that they are a
''universal'' means of storing data in an application-independent
manner, just as XML is a universal syntax for communication of data. SQL
attempts to be "the" universal language for querying from and
manipulating such data in relational databases; unfortunately it has not
been very well standardized.

To a mathematician (and I hope any computer programmer), the "logic of
sets" presents no difficulties as soon as one understands the
mathematical definition of a "table". From this perspective, a
relational database could be regarded as a category of "relations". A
"type" is a set (in the standard sense), such as the set of integers or
the set of finite strings over an alphabet. A "tuple" is an element of a
cartesian product (as sets) of types. A "relation" is a set of tuples
(this has nothing to do with the notion of a binary or n-ary relation
from mathematics). A relation is informally regarded as a "table",
although this is only correct if one regards the rows (i.e. tuples) of
the table as being unordered, and if one disregards the column labels.
The projections of tuples to their various factors determine
"attributes" of the tuples or "fields" of the tables, and one usually
regards these factors as labeled with convenient names ("column
headers").

The cartesian product ("cross join") of tables is the usual cartesian
product of the corresponding relations as sets. An "inner join" is a
certain subset (diagonal) of a cartesian product of tables: if one has
tables S and T and S has attributes i and T has attributes j then the
inner join asks for the subset of S x T where s.i = t.j for some i's and
j's. To explain the "outer" joins, one must introduce NULL tuples
compatible with the respective product structures of S and T, so that
one is taking an appropriate subset of S x (T u NULL) or (S u NULL) x T
respectively. Then the corresponding elements of the S x NULL or NULL x
T subsets are used whenever a record does not satisfy the conditions
"on" the attributes, i.e. when s.i = t.j fails for some pair (i, j).

I have amused myself by thinking about such details, and in one case it
had a practical consequence: I realized that a join could be empty in
the case that at least one of the relations/tables in the join was
empty, and that this consideration required me to write code that
*always* generated at least one (possibly NULL) tuple in the result-sets
that were to be joined. Further, it is useful to know that the cross
join of two or more one-element tables is again a one-element table.
This in turn is occasionally handy as a very simple pivoting technique
for the presentation of aggregates with column labels. And so on... it
is fun to think of what various other categorical constructions will
give. This is how a mathematician keeps himself mentally occupied with
SQL.
