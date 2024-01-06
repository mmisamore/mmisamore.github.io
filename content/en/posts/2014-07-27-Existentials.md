---
title: Adventures in Existential Quantification 
date: 2014-07-27
tags: 
  - existential quantification
---

I recently found [this post](http://robots.thoughtbot.com/thinking-in-types)
about dealing with `Renderable` objects in the context of game design. The
solution presented there introduced a composite type

```haskell
data Game = Game Ball Player Player
```

where `Ball` and `Player` are instances of a typeclass `Render a`:

```haskell
class Render a
where render :: a -> IO ()
```

Then one can write a function that renders a `Game` by pattern matching on the
`Game` constructor. If the constructor for `Game`s changes, so must the `Render`
implementation for `Game`. This is not ideal, so one might try to use a composite
pattern like 

```haskell
data ExtendedGame = ExtendedGame Game Score 
```

to extend `Game`, but then functions that consumed `Game`s cannot automatically
consume `ExtendedGames` even if they only require `Game` functionality. Creating
dependencies on constructors like this seems like a bad idea. 

This approach to rendering was introduced because otherwise one would have to
deal with rendering heterogeneous lists like `[Ball, Player, Player]`.  

```haskell
ghci -XExistentialQuantification
> class Render a where render :: a -> IO () 
> data Player = Player
> instance Render Player where render p = print "player"
> data Ball = Ball
> instance Render Ball where render b = print "ball" 
```

Defining a rendering function like

```haskell
renderAll :: forall a. (Render a) => [a] -> IO ()
renderAll = mapM_ render
```

fails to do the job because the type `[a]` still constrains the input to a
*single* instantiation of the type variable `a`. This can be fixed by using an
existential type:

```haskell
> data Renderable = forall a. (Render a) => Renderable a
```

In other words, we can inject any `a` implementing `Render` into a `Renderable`. So
much for the constructor, but it would also be nice if `Renderable`s could be
rendered!

```haskell
> instance Render Renderable where render (Renderable a) = render a
```

Finally we can write the generic renderAll function:

```haskell
> :m Data.Foldable -- for sequenceA_ 
> let renderAll :: [Renderable] -> IO (); 
  renderAll = sequenceA_ . map render 
> renderAll [Renderable Player, Renderable Ball, Renderable Player]
"player"
"ball"
"player"
```

So now we can generically render any list of `Renderable`s, and any `Game` should be
able to provide us with a list of `Renderable`s:

```haskell
> let gameRenderables :: Game -> [Renderable]; 
  gameRenderables = -- ...
```

At this point we don't care about how `Game`s are constructed; we just care that
they can provide lists of `Renderable` objects. So we also have a totally generic
way to render a `Game`:

```haskell
> instance Render Game where render = renderAll . gameRenderables
```

`Game` instances can then provide greater or fewer `Renderable`s based on their
states - they are not tied to their constructors. Furthermore, the `Render`
instance for `Game` does not change if more `Renderable`s are introduced later on.

Now there is a lot of juicy discussion over
[here](http://lukepalmer.wordpress.com/2010/01/24/haskell-antipattern-existential-typeclass)
regarding why *not* to use existentials in this way. In that example, one is
using a typeclass `Widget` when a `Widget` is really an *object*, not a
*behavior*! As for `Renderable`, the argument is to use data types

```haskell
> data Renderable = Renderable { render :: IO () }
```

instead so we just capture the desired IO actions. Instead of instances we have
functions:

```haskell
> let toRenderable i = Renderable (print i)
```

This takes any Showable and turns it into a `Renderable`, for example. Supposing I
have a composite object (like `Game`) full of `Renderable`s, I can sequence their `IO`
actions in the desired order and produce any desired additional `IO` to generate a
new `Renderable`. For example:

```haskell
> let listRend :: [Renderable] -> Renderable; 
  listRend = Renderable . mapM_ render
```

We generate a new `Renderable` by sequencing the renderings of the `Renderable`s in
the list. There is still a nice typeclass pattern here: `toRenderable` is a
behavior that takes a type and returns its corresponding `Renderable`. In other
words, we can **defer the behaviors to an intermediate type which allows us to
manipulate and sequence them in homogeneous containers**.

```haskell
> class Render a where toRenderable :: a -> Renderable
> instance Render Player where toRenderable p = Renderable $ print "player"
> instance Render Ball where toRenderable b = Renderable $ print "ball"
> render $ toRenderable Player
"player"
> render $ listRend [toRenderable Player, toRenderable Ball, toRenderable Player]
"player"
"ball"
"player"
```

This is all still completely statically type-checked. Yay! :-)

