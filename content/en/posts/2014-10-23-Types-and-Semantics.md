---
title: Types and semantics 
date: 2014-10-23
tags: 
  - types
  - semantics
---

If we think of a "personName" as a possible name for a person, then semantically
it carries more information than just being an arbitrary String. Arbitrary
strings are, well, arbitrary! Even if we really thought that people could have
arbitrary strings as their names, the fact that they *are names* (meaning that
they are intended to be *used* as names) means that they should have a type
separate from String, say PersonName. 

Indeed, we expect that the behavior of a PersonName could be different than the
behavior of a String: it might have a canonical decomposition into components
like FirstName/LastName, it might admit various translations, or it might have
constraints on the allowable character sets that are not present for Strings.
The fact that we could use a String as an underlying *representation* of a
PersonName is beside the point: it violates information-hiding to use String as
the actual type. 

This is also relevant with respect to function parameters: a function of type

```haskell
String -> String -> String -> Int -> String -> IO ()
```

doesn't tell me a whole lot about what the parameters are, and the danger of
confusing the ordering results in language workarounds like named parameters.
Instead of naming the parameters, why not just give them more descriptive types
in the first place and get more type safety for free? 

Another canonical example of this problem that shows up everywhere is using Int
for nearly every numeric identifier (aside from floating point). Did you really
want to include negative numbers, and if not, then *why did you include them?*
Part of the point of using types is to simplify the tasks of downstream
functions: if I know the input is a natural number then in particular I never
have to worry about it being negative, and the compiler has my back. This makes
me feel warm and fuzzy when trying to reason about my code.

