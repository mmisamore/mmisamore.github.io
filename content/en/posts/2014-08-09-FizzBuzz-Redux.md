---
title: Fizzbuzz Redux 
date: 2014-08-09
tags: 
  - fizzbuzz
---

There is an interesting monoid instance for `Maybe a` types whenever
`a` is a monoid, which ignores `Nothing` and applies the underlying
monoid under the `Just`. In Fizz-Buzz, Fizz is supposed to appear in place
of every third number, and Buzz in place of every fifth number, but both should
appear on every fifteenth number. So we can generate a fizzbuzz stream first,
and then find a way to combine it with integers. The key is to combine with
*another* monoid structure on `Maybe a` called the "alternative", which
exists since `Maybe` is applicative. This latter monoid selects the first
`Just` value that appears in a product, or is `Nothing` if no such value
is found. Zipping the fizzbuzz stream against the list of numbers with the
alternative operator `<|>` gives a list of "just" the preferred values, and then one
can safely filter for only the Justs and monadically print the list:

```haskell
:m +Data.Monoid
:m +Control.Applicative
:m +Data.Maybe
let fizzStream = cycle [Nothing, Nothing, Just "Fizz"]
let buzzStream = cycle [Nothing, Nothing, Nothing, Nothing, Just "Buzz"]
let fizzBuzzStream = zipWith (<>) fizzStream buzzStream

let fizzBuzz = mapM_ putStrLen . catMaybes $ 
               zipWith (<|>) fizzBuzzStream (map (Just . show) [1..100])
```

What I like about this solution is that it consists entirely of independent,
composable pieces, and there is no conditional logic in sight. The monoid
structure *encodes* the conditions of preference. This is certainly not my idea
(I think I first heard about it on reddit), but it's quite slick so I wanted to
implement it again myself.

