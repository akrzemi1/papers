Anthony's example with contracts: mandate compiler error on `f(-1)` (core constant expression).

Static analysis detects bugs, but it is only a warning -- and we live with it.

```c++
template<int N> void f() {
  if (N > 0) {
    const auto f2 = impl(N);
  } else {
    // other stuff
  }
}
```

Should a call to f<0>() be ill-formed? I don't think so. This seems best left to QoI to me (a compiler can choose to warn on this only if it believes the call to impl(N) is actually reachable, for instance).
