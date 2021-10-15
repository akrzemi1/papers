### Test sequence

Given a list of preconditions:

```c++
int f(int i, int const j)
  [[pre: a(i)]]
  [[pre: b(j)]];
```

A precondition test sequence is a function defined like this:

```c++
[](int& i, int const& j) noexcept
{    
  if (PRED(a(i))) {} else {
    CONTRACT_VIOLATION_HANDLER();
    std::abort();
  }

  if (PRED(b(j))) {} else {
    CONTRACT_VIOLATION_HANDLER();
    std::abort();
  }
}
```

Where:
 
 * Parameters are references and are constructed from function parameters by adding an lvalue reference.
 * Expressions in precondition predicates are looked up as if they appear in `noexcept` specifications of the functions they appertain to.
 * `PRED(expr)` is either expression `expr` or an expression that returns the same `bool` value as `expr` but has no side effects.
 
 
### Evaluation order

  1. initialize function parameters (including default function arguments)
  2. **perform precondition test sequence** (inspecting function argument objects)
  3. start function call (noexcept barrier)
  4. **perform precondition test sequence** (inspecting function argument objects)
  5. run function body
  6. initialize the return object
  7. call destructors of automatic objects
  8. **perform postcondition test sequence** (inspecting function argument objects, and return object)
  9. end function call (noexcept barrier)
 10. **perform postcondition test sequence** (inspecting function argument objects, and return object) 
 11. call destructors of function parameters
