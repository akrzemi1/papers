Random notes on contracts
=========================

do we want same student's experience for [[assert: c]] and language UB for pointer dereference (through UB-sanitizer?

asserts that act like preconditions

by introducing preconditions to my code, I am potentially introducing bugs, as the prdicates can have UB. The goal of disabling assertions in production may be to avoid bugs

substitutability: one function can meet two different contracts (measured as pair of pre and postcondition)


If I wat to wrap my `std::array<int, a>` into a non-template class/function, I may want `[[pre assume]]` to guarantee the same compiler optimizations as before the wrap.

Document tree precondition use-cases.

Document use case for `__builtin_assume()`.

Document time travel optimizations.

Static levels will never be sufficient:

```c++
template <typename T>
void f(T x, T y) [[pre LEVEL: X != y]];
```

The level may depend on `T`.

Is it correct to test the postcondition if precondition failed? How does Bloomberg testing handle that?

```c++
fun(ptr); // template with defensiv if
gun(ptr); // has precondition ptr != null
```

We want the defensive if in fun() to be removed.

```
bool is_open(P * p) [[pre: p]];

void f(P * p)
  [[pre: p]]
  [[pre: p->is_open(p)]];
```

One of he checks can be elided. Also: can predicates have narrow contract?
