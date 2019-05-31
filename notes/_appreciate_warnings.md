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


Compiler should have the right to ignore all attributes -- this may be the criterion for choosing the attribute.

Recomend a standalone paper with recommended warnings.

`try`-expression -- if it is only a recomended warning, compiler vendors will be allowed to implement heuristics when to warn and where to avoid false positives or technical problems like surprising template instantiations.

If vendors are allowed to implement heuristics there would not be a prblem suc as surprise template instantiation when determining narrowing.

Feature test for determining if compiler implements semantics recommended for a given attribute.

```c++
template <typename T>
struct Value { static T instance; };

template <typename T>
constinit Value<T> Registry<T>::instance;
```

-- user of the library template does not even know that I am using a static member and `constinit` inside. I do not know that user will give his type so I cannot test it in my compiler. Therefore I need a syntax rather than semantics.
