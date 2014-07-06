---
layout:     post
title:      "Debug Logging in C"
---

Using `printf` for debugging is simple enough for small projects, but what if
we want more information?


## Beginnings

We start off by defining a simple `LOG` macro to essentially alias `printf`,
with a simple way to toggle messages:

{% gist abstractOwl/e512ce07925e5e38b08e simple_log.h %}


## Adding Debugging Levels

Okay, so how about separate debugging levels? Traditionally, loggers print
messages from the lowest defined level on up. Here we'll use `NONE`, `INFO`,
and `ERROR`, but you can add your own. This system can be implemented by simply
adding a few lines:

{% gist abstractOwl/e512ce07925e5e38b08e tiered_log.h %}


Using preprocessor constants and conditionals, we can construct a more robust
tiered logging system.

There are several interesting additions in this iteration. First, we see that
the strings "[WARN ]" and "[ERROR]" are prepended directly to msg; there is no
need for `+` or other operators to signify the concatenation.

On the same line is `## args`. Consider the case where we run
`LOG("Hello World")` with `LOG` from the first iteration. Because the varargs
are empty, we would essentially be writing `printf("Hello World", )`, causing a
syntax error. Thus, this extension works around that issue and makes varargs
optional. **Note that this is a GCC extension and may not be supported by all
compilers**. You can read more on this in the official GCC documentation[^1].


## Printing Source Code Metadata

One of the most frustrating aspects of debugging is not knowing where messages
are coming from. Of course, there's always wading through `grep` or one of its
self-proclaimed successors ([Ack](http://beyondgrep.com) or
[Ag](http://betterthanack.com)), but there has to be an easier way.

The C preprocessor conveniently provides the `__FILE__`, `__LINE__`, and
`__func__` macros to help with this dilemma. These macros can be put to quick
use by factoring logging logic into a separate `LOG` macro:

{% gist abstractOwl/e512ce07925e5e38b08e logc.h %}

(Props to StackOverflow for help on  preprocessor variables[^2]).


## Result

At this point, our library provides logging levels and extra source code
metadata to help with debugging.

Simple, effective logging is crucial, especially when building complex systems
and triaging bugs. You can grab the final code in Gist form
[here](https://gist.github.com/abstractOwl/e512ce07925e5e38b08e).

Questions or suggestions? Start a discussion in the comments below!


## References

[^1]: [https://gcc.gnu.org/onlinedocs/cpp/Variadic-Macros.html](https://gcc.gnu.org/onlinedocs/cpp/Variadic-Macros.html)
[^2]: [https://stackoverflow.com/questions/1644868/c-define-macro-for-debug-printing/](https://stackoverflow.com/questions/1644868/c-define-macro-for-debug-printing/)

