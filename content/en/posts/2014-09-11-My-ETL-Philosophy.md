---
title: My ETL philosophy 
date: 2014-09-11
tags: 
  - sql
  - databases
---

The term ETL stands for Extract, Transform, and Load. In a database it is very
common to **E**xtract data from multiple sources (interior or exterior to the
local database), **T**ransform that data in some prespecified manner, and finally
**L**oad that transformed data into a destination resource such as a table. In
SQL on relational databases, it seems to be common practice to have stored
procedures handle all three tasks: extracting data from multiple tables or other
resources, transforming it (often via temp tables or table variables), and
finally dumping the results into destination tables.

My philosophy, which is inspired by the no-side-effects school of thinking, is
to recognize that two of these steps, namely Extract and Transform, are
**read-only**, so that most of the heavy lifting can be accomplished without
affecting the state of the database (ignoring read locks). In MS T-SQL this can
be implemented with user-defined functions, which are trivial to create, test,
and modify, because they *do not affect the state of the database*. If I see a
user-defined function I know immediately that I can run it, even on a
production system, with very little risk of breaking anything. Can you say the
same thing about running a random stored procedure in your database?

What's amazing about this approach is that user-defined functions are reusable:
you can easily query off of the results of table-valued user-defined functions,
store those results in common table expressions or table variables, combine
those results with queries on other user-defined functions, and so on. With
sufficiently careful coding (indices on tables and table variables) these can be
reasonably fast, even when nesting functions several levels deep. The stored
procs which do the **L**oading are then thin wrappers around mostly read-only
extractions and transformations of data. 

When this approach works, it works really well. Instead of running effectful
stored procedures when you have to debug, you are running read-only functions
which are trivial to break down to their underlying read-only queries. Even when
deleting records, you can determine the primary keys via a read-only process and
then test that it's working as expected before executing any actual deletion.
The major exception to the applicability of this approach comes from the
performance side: SQL databases seem to be optimized to efficiently execute
stored procedures, not functions, and it's likely that uglier methods like temp
tables will have better performance. But when performance is not the primary
concern (but rather **correctness**), it begins to make a lot of sense to
consider such an approach.

