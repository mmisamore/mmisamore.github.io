---
title: Yesod and cabal hell 
date: 2014-06-09
tags: 
  - haskell
  - cabal
---

Today I decided to have a go at installing Yesod for the first time. I already
had the Haskell Platform installed, so according to the
[tutorial](http://yannesposito.com/Scratch/en/blog/Yesod-tutorial-for-newbies)
I was supposed to run

```bash
cabal update
cabal install yesod yesod-bin cabal-dev
```

and be on my merry way. Unfortunately, instead of having a functional Yesod
installation I had instant Cabal hell. Doing `yesod init` set up the scaffolding
fine, and it recommended a `cabal sandbox init` so I did that, but then 
`yesod devel` fed back messages like these:

```bash
Configuring yesod-test-0.0.0...
cabal: At least the following dependencies are missing:
conduit >=1.0 && <2.0,
data-default -any,
fast-logger >=2.1.4 && <2.2,
hjsmin ==0.1.*,
http-conduit ==2.1.*,
monad-control ==0.3.*,
monad-logger ==0.3.*,
persistent ==1.3.*,
persistent-sqlite ==1.3.*,
persistent-template ==1.3.*,
shakespeare ==2.0.*,
wai-extra ==2.1.*,
wai-logger ==2.1.*,
warp ==2.1.*,
yaml ==0.8.*,
yesod >=1.2.5 && <1.3,
yesod-auth ==1.3.*,
yesod-core >=1.2.12 && <1.3,
yesod-form ==1.3.*,
yesod-static ==1.2.*
```

Wha? I thought I just installed yesod, so why is it asking me to do it again?
I can understand that it's a sandbox so I suppose there is no look-through to
the global package db. Makes some sense.

Anyway, trying to `cabal install` these manually in the sandbox ran me into a
wall with `network-2.5.0.0`, which refused to compile in the sandbox but compiled
fine outside it (wtf?). Okay, deleting the sandboxed cabal file and installing
`network-2.5.0.0` in the global package db seemed to work. At this point I was
about 5 hours into getting this stuff to work after multiple reinstalls of
Haskell Platform, blowing away `~/.ghc`, upgrading cabal versions, etc. Getting
tired of waiting for my machine to compile several hundred dependencies... this
is why I stopped using Gentoo Linux - **just give me stuff that works, please.**

Okay, so now I [remembered](http://www.yesodweb.com/page/quickstart)
yesod-platform. Will try it first with a sandbox, then without. No dice with
sandboxed version after about 30 minutes (tons of install failures having to do
with `network-2.5.0.0` again - clearly a real issue).  Dropped sandbox and now
running canonical command:

```bash
cabal install yesod-platform yesod-bin --max-backjumps=-1 --reorder-goals
```

Well, it finished the install without errors this time. Running `yesod init`
gives me the message

```bash
Start your project:

   cd yesod-test && cabal sandbox init && cabal install --enable-tests .
   yesod-platform yesod-bin --max-backjumps=-1 --reorder-goals && yesod
   devel
```

but I've learned that this command leads back to perdition... so skipping ahead
to the `yesod devel` part: 

```bash
cabal: At least the following dependencies are missing:
persistent-postgresql ==1.3.*, wai-extra ==2.1.*, warp ==2.1.*
```

I chose to use postgresql, so I'm not that surprised by that one, but missing
warp too? Seems odd since yesod usually runs on Warp, but whatever. Trying to
install these manually yields... okay, `persistent-postgresql` was not a problem,
but yesod wants older versions of `wai-extra` and `warp`. Looking up on hackage
for anything in warp's `2.1.*` family led me to:

```bash
cabal install warp-2.1.5.2
```

Now I'm left with `wai-extra`. Hackage tells me that the latest is `3.0.0`; not so
lucky with installing `2.1.*`:

```bash
$ cabal install wai-extra-2.1.1.2
Resolving dependencies...
In order, the following would be installed:
wai-logger-2.1.1 (reinstall) changes: wai-3.0.0 -> 2.1.0.3
zlib-conduit-1.1.0 (new package)
wai-extra-2.1.1.2 (latest: 3.0.0) (new version)
cabal: The following packages are likely to be broken by the reinstalls:
yesod-platform-1.2.12
yesod-core-1.2.16
yesod-test-1.2.3
...
Use --force-reinstalls if you want to install anyway.
```

So it's telling me that if I want `wai-extra` I have to break Yesod, or I could
`force-reinstalls` and risk breaking everything. Hmm... well, nothing to lose at
this point.

```bash
cabal install wai-extra-2.1.1.2 --force-reinstalls
```

and... success? Now I have to check that I didn't break anything, and I did
(`yesod devel` is complaining about broken dependencies now).  Then I attempt to
`--reinstall` and `--force-reinstalls` on some packages as a last-ditch fix, but
that seems to be going nowhere.

Well, that was one full workday attempting to install Yesod. I think that's
enough time invested for the next year or so. Perhaps by then the install issues
will be smoothed out. It's a shame because I thought the platform was otherwise
really attractive. I hope that by spelling things out a bit here it can show how
difficult/frustrating the process can be.

