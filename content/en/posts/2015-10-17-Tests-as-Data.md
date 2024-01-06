---
title: Tests as Data 
date: 2015-10-17
tags: 
  - testing
  - purity
  - free monads
  - monoids
---

Currently, most popular approaches to software testing used by developers
consist of unit-testing frameworks such as NUnit/JUnit and the like. Putting the
[debate between mockist and classicist
testing](http://martinfowler.com/articles/mocksArentStubs.html#ClassicalAndMockistTesting)
aside for the moment, I think it's useful to step back and ask ourselves what
tests are, and why we have evolved into these testing patterns. To me, a test
consists of an evaluation of a system or subsystem to ensure that it satisfies
some specification. Following the principle of parsimony, a test should
therefore resemble the specification as closely as possible, and in particular it
should not be tied to the software implementation if that can be avoided.

In unit-testing frameworks, we end up with a bunch of test functions/methods
which provide an *executable* evaluation of system behavior, and in doing so we
have immediately bound ourselves to a particular implementation of our tests,
bugs and all. If the test (i.e. evaluation) changes, we are then forced to
either a) directly update our test implementations, potentially introducing new
bugs in our tests; or b) update some middleware specification such as a Gherkin
script in Cucumber. While this middleware approach goes a ways towards solving
this problem, one usually (e.g. in Cucumber) binds the individual steps in test
scripts back to a single fixed implementation. A couple of questions arise: 1)
why are we storing these scripts in Visual Studio/Eclipse where our testers
cannot easily access/run them? 2) Would it be useful to have more than one
implementation of a given test?

I have drawn a couple of conclusions from the above considerations: the first is
that automated tests, being evaluations, should not have side-effects.
**Automated tests are therefore better represented as pure data** rather than
functions/methods.  The second is that, **if tests are data, then it could be
really useful to have multiple interpreters** for tests: the two that immediately
come to mind are "describe this test" and "run this test". Others could include
"run this test and generate a report in HTML" or "run this test and store the
test results in a db". Some of these implementations are pure functions (in the
functional programming sense), and some are not, but we are not forced into
using effectful interpretations if we don't want them.

If a test is data, then what does it consist of? Well, the obvious model is that
a test is some kind of list of steps. But lists are monoids, so we can
immediately concatenate lists of steps/tests together to yield larger tests. If
a test includes a step that an interpreter doesn't understand, the interpreter
can take some action that is decoupled from the test itself: perhaps it just
throws a warning, or perhaps it fails. We can use arbitrary list transformations
on our tests to build new tests: filtering/replacing steps, repeating steps,
embedding other tests, etc. We can even write separate interpreters to ensure
that we are getting consistent results between implementations.  

Finally, we can serialize/deserialize the tests themselves to different formats
via interpreters.  If you want to serialize a test to a readable Cucumber-like
specification that you can pass to a product owner, go ahead. If you want to
communicate your tests via JSON, you can do that too. Store your tests in a
database if you feel like it.

So, I've been playing with writing a DSL that implements these ideas with
interpreters for free monads. Thus far it seems pretty successful and I haven't
run into any major hurdles, and it feels considerably more flexible than other
approaches I've seen to automated testing.

