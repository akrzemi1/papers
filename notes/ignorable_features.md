Ignorable features
==================

This paper is an attempt to structure the discussion on the choice of syntax for contract annotations.

[P0542](https://wg21.link/p0542) and [P2388](https://wg21.link/p2388r2) propose a syntax that appears
similar to attributes:

```c++
int f(int i)
  [[pre: i >= 0]]
  [[post r: r >= 0]];
```

[N1962](https://wg21.link/n1962) and [P2461](https://isocpp.org/files/papers/P2461R0.pdf) propose an
alternate syntax, nothing like attributes:

```c++
int f(int i)
  pre{i >= 0}
  post(r){r >= 0};
```

Interestingly, the argument brought up in favour of the attribute-like syntax is that contract annotaitons
are "ignorable"; whereas the arguments against the attribute-like syntax say that contract annotations are 
not "ignorable". This indicates that name "ignorable" is overloaded and requires a clarification.


"Ignorable"
-----------

We can get the best approximation from the C Standard ([N2596](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2596.pdf)), which defines what it means for a standard attribute to be ignorable.

6.7.11 p2:

> Support for any of the standard attributes specified in this document is implementation-defined
> and optional. For an attribute token (including an attribute prefixed token) not specified in this
> document, the behavior is implementation-defined. Any attribute token that is not supported by the
> implementation is ignored.


4 p5:
> A strictly conforming program shall use only those features of the language and library specified
> in this document. It shall not produce output dependent on any unspecified, undefined, or
> implementation-defined behavior, and shall not exceed any minimum implementation limit.

6.7.11.1 p3:
>  [...] A strictly conforming program using a standard attribute remains strictly conforming in the absence of that
> attribute. <sup>166)</sup>

Footnote 166:
> Standard attributes specified by this document can be parsed but ignored by an implementation without changing the
semantics of a correct program; the same is not true for attributes not specified by this document
