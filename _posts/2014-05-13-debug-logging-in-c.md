---
layout:     post
title:      "Debug Logging in C"
---

Using `printf` for debugging is simple enough for small projects, but what if
we want more information?


## Beginnings

We start off by defining a simple `LOG` macro to
alias `printf`, with a simple way to toggle messages:

{% gist abstractOwl/e512ce07925e5e38b08e simple_log.c %}


## Adding Debugging Levels

Okay, so what if we want separate debugging levels? Here we'll use NONE, INFO,
and ERROR, but you can add your own. Traditionally, loggers print messages 
from the lowest defined level on up. We can implement this system simply
by adding a few lines:

{% gist abstractOwl/e512ce07925e5e38b08e tiered_log.c %}


Using constants and conditionals, we can construct a more robust tiered logging 
system.

There are several interesting additions in this iteration. First, we see that
the strings "[WARN ]" and "[ERROR]" are prepended directly to msg; no need for
`+` or other operators.

Shortly following it in `printf` is `## args`. **Note that this is a GCC
extension and may not work in all compilers**. Consider the case where we run
`LOG("Hello World")` with `LOG` from the first iteration. Because the varargs
are empty, we would essentially be writing `printf("Hello World", )`, causing
a syntax error. You can read more on this in the official GCC documentation[^1].


## Printing Source Code Metadata

One of the most frustrating aspects of debugging is not knowing where messages
are coming from. Of course, there's always wading through `grep` or one of its
self-proclaimed successors ([Ack](http://beyondgrep.com) or
[Ag](http://betterthanack.com)), but there has to be an easier way.

The C preprocessor conveniently provides the `__FILE__`, `__LINE__`, and
`__func__` macros to help with this dilemma. We can easily insert this into
the previous iteration with an extra macro `WHERE`:

{% gist abstractOwl/e512ce07925e5e38b08e logc.c %}

(Props to StackOverflow for help on  preprocessor variables[^2] and concatenating
 macros[^3]).


## Result

Simple, effective logging is crucial, especially when building complex systems
and triaging bugs. You can grab the final code in Gist form
[here](https://gist.github.com/abstractOwl/e512ce07925e5e38b08e).

Questions or suggestions? Start a discussion in the comments below!


## References

[^1]: [https://gcc.gnu.org/onlinedocs/cpp/Variadic-Macros.html](https://gcc.gnu.org/onlinedocs/cpp/Variadic-Macros.html)
[^2]: [https://stackoverflow.com/questions/1644868/c-define-macro-for-debug-printing/](https://stackoverflow.com/questions/1644868/c-define-macro-for-debug-printing/)
[^3]: [https://stackoverflow.com/questions/19343205/c-concatenating-file-and-line-macros](https://stackoverflow.com/questions/19343205/c-concatenating-file-and-line-macros)
