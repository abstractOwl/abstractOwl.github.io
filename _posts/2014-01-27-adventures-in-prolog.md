---
layout:     post
title:      "Adventures in Prolog"
---

Over the past couple days, I've been banging my head against the wall, trying to
understand the logic programming language
[Prolog](https://en.wikipedia.org/wiki/Prolog) (or more specifically,
[SWI-Prolog](http://www.swi-prolog.org/)) for our CS162 Programming Languages
class.


<br />

## What is Prolog?

**Prolog** is a declarative programming language. It operates by performing
queries against given *facts* and *relations*, somewhat similar to database
query languages like SQL. An example of Prolog:

        man(adam).
        man(peter).
        woman(alice).
        woman(mary).
        woman(sara).
        parent(adam, peter).
        parent(adam, mary).
        parent(adam, sara).
        parent(alice, peter).
        parent(alice, mary).
        parent(alice, sara).

        father(F, C) :- man(F), parent(F, C).
        mother(M, C) :- woman(M), parent(M, C).
        siblings(A, B) :- parent(P, A), parent(P, B), A \= B.

This example illustrates a family of 5; Adam and Alice have three children,
named Peter, Mary, and Sara.

In Prolog, identifiers beginning with lowercase letters are known as atoms or
*constants*. `adam` is an atom.

Identifiers beginning with capital letters are *variables*. `F`, `M`, `C` are a
few of the variables in this example. Note that variables can be more than one
letter, like `Dog`.

*Functions* in the language are known as predicates; they take one or more
parameters and return a boolean value. In the above example, 

In the above example, `father`, `mother`, and `siblings` are obviously
predicates. But notice `man`, `woman`, and `parent`. These *are also*
predicates; `man(adam).` signals that when the predicate `man` is used with atom
`adam`, it will always return true.

What's interesting is that predicates can be called with an uninitialized
variable. In this case, Prolog conducts a search for all possible values for
that variable that cause the predicate to be true. For example, the query:

        ?- siblings(peter, Siblings).

returns:

        Siblings=[mary, sara].

To programmers more familiar with the imperative style, Prolog presents a
vastly different perspective.

<br />

## More Resources

Some resources I found useful for learning Prolog:

* [Learn Prolog Now!](http://learnprolognow.org/) -- Great tutorial for
      getting started.
* [SWI-Prolog Manual](http://www.swi-prolog.org/pldoc/doc_for?object=manual)
      -- SWI-Prolog manual pages provide a quick reference for looking up
      functions and parameters.
* [StackOverflow](http://www.stackoverflow.com) -- As with anything else
      programming related, StackOverflow is here to help.

