---
layout: post
title:  "UINT_MAX in C"
---

**Quick tip**: You can also get `UINT_MAX` in C by doing `(unsigned) -1`.

Why does this work? In C, negative numbers are stored in
[Two's complement](https://en.wikipedia.org/wiki/Two's_complement)
representation.

In this representation, signed numbers use the highest bit, the sign bit, to
store the sign of the number. Negation is done by flipping all bits and adding
1. In unsigned 4-bit numbers, 1 would look like `0001`, while -1 would be
`1110 + 1 = 1111`, the maximum 4-bit number.

[This StackOverflow question](http://stackoverflow.com/a/50632) notes
that C believes unsigned 1 `0001` *is less than* signed -1 `1111`, again due to
the internal representation.
