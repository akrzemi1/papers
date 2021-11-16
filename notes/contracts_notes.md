Random notes on contracts
=========================







substitutability: one function can meet two different contracts (measured as pair of pre and postcondition)


If I wat to wrap my `std::array<int, a>` into a non-template class/function, I may want `[[pre assume]]` to guarantee the same compiler optimizations as before the wrap.

Document tree precondition use-cases.



```c++
fun(ptr); // template with defensiv if
gun(ptr); // has precondition ptr != null
```

We want the defensive if in fun() to be removed.

```c++
bool is_open(P * p) [[pre: p]];

void f(P * p)
  [[pre: p]]
  [[pre: p->is_open(p)]];
```

One of he checks can be elided. Also: can predicates have narrow contract?



