---
layout:     post
title:      "`void` keyword as function parameter"
---

While wading the Linux source code (specifically `arp.c`), I noticed that
several function declarations used the `void` keyword as a parameter, like:

        static int arp_proc_init(void);
        void __init arp_init(void);

According to [StackOverflow](https://stackoverflow.com/questions/51032/is-there-a-difference-between-foovoid-and-foo-in-c-or-c), using `void` in this way
achieves "consistent interpretation of headers that are shared between C and
C++".


In **C**:

> `void foo()` means "a function `foo` taking an unspecified number of
> arguments of unspecified type"
>
> `void foo(void)` means "a function `foo` taking no arguments"


In **C++**:

> `void foo()` means "a function `foo` taking no arguments"
>
> `void foo(void)` means "a function `foo` taking no arguments"


In short, this usage of the `void` keyword specifies an empty parameter list in
files which will be used in C.
