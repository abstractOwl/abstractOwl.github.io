---
layout: post
title: Barebones Unit Testing In C
---

We all know those nights spent pounding away at a personal project. Let's face
it, unit testing in C is a pain in the butt. If you can't be bothered to pull
[Check](http://check.sourceforge.net/),
[ATF](https://code.google.com/p/kyua/wiki/ATF), or the like into your project,
[Jera Design](http://www.jera.com/techinfo/jtns/jtn002.html)
presents an *extremely* simple testing framework -- MinUnit[^1]:

    /* file: minunit.h */
    #define mu_assert(message, test) do { if (!(test)) \
        return message; } while (0)
    #define mu_run_test(test) do { char *message = test(); \
        tests_run++; if (message) return message; } while (0)
    extern int tests_run;

*... and that's all folks!*

**Notes:**

[^1]: Had to rewrap the code slightly to get it to fit into the rectangle.
