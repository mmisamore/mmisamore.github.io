---
title: Vim, Forth, and Emergent Behavior 
date: 2014-03-15
tags: 
  - vim
  - forth
  - editing
---

As an oddball way of "updating" my skills, I decided some 6 months ago to learn
Vim. Of course, I had to debate over Vim vs. Emacs and other editors first, but
I had used Emacs and relatives before and found their Ctrl/Meta-everything
approach to keyboard shortcuts to be annoying. On the other hand, I had also had
the classic Vim newcomer experience: you start Vim, and are immediately
terrified to find that you have no idea how to do anything or even quit the
editor! :-)

There were a couple of annoyances at first: first and foremost, hitting `Esc` to
get back to normal mode is a ludicrous excursion from the home row. Fortunately
this was very easy to fix, and  

```bash
imap ;; <Esc>
```

in my .vimrc did the job of giving me a *very* fast home row escape. Second, the
Vim documentation is fairly intimidating when you first read it - it's a bit
disingenuous to complain about the complexity of Emacs when Vim is rather
complex itself, and its "Vim Script" language is nothing to brag about.

What I do like about Vim is how simple it is to learn and use: Vim recognizes
that most of the time you are either 1) navigating through a document or 2)
transforming certain blocks of text within the document, so the commands mostly
focus on making navigation and manipulation easier. Most commands are either a
single character or two characters, and many commands can be repeated by
prefixing with a count. If I want 10 lines containing "Hello, world!", it's a
matter of typing

```bash
10i"Hello, world!"<CR>;;
```

If I want a list of numbers from 1 to 10, it's just:

```bash
:put =range(1,10)<CR>
```

If I want to append a comma to each resulting line, it's

```bash
Ctrl-V}$i,;;
```

which basically means to enter visual block mode, navigate to the end of the
current "paragraph", move to the end of the line, and insert a comma on all
selected lines. If I want to join the resulting lines, I can navigate back to
the top with `{` and then `10J`, where `J` is the command to "join" lines together.

There are several other awesome commands, like `g;` which navigates to the last
change (extremely handy for cross-referencing code), `gqap` and `gqip` for
quickly reformatting paragraphs, `d` for deleting along certain movements (`2dw`
deletes the following two words and `2d)` deletes the following two sentences),
`f` for moving forward along a line to the first occurence of the following
character (which search can be repeated with `;`), and finally the venerable `q`
which allows one to record macros (which are literally stored as ordinary
commands given by sequences of characters) and its counterpart `@` which plays
back macros and can be prefixed with a count to play a given macro a specified
number of times.

What emerges out of this is a small set of very fast, reusable, repeatable
commands for editing text. Together with the macro feature it is very much like
a "Forth" approach to editing: provide a very simple set of "words" or
"commands" to perform basic tasks, and then combine/replay them to achieve more
complex behavior. The learning curve is very fast: it usually takes less than
a minute to learn a new word/command, and one can immediately start combining
it with other commands to more easily achieve the desired behavior. Since there
are not many core commands to learn, muscle memory soon takes over to make
editing faster relative to more naive editing approaches.

What would make this ideal for editing code is integration with more modern
IDEs. There are plugins like "haskellmode" available, but preferably editors
would just "drop into" existing IDE platforms (even stuff like TeXworks could
benefit from this). What is the right model for fully decoupled text editor/IDE
interaction? 

