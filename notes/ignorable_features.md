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
