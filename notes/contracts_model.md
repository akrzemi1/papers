### Test sequence

Given a list of preconditions:

```c++
int f(int i, int const j)
  [[pre: a(i)]]
  [[pre: b(j)]];
```

A precondition test sequence is a function defined like this:

```c++
void pre_test(int& i, int const& j) noexcept
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
 
Note: These parameters may be references to mutable objects. While modifications of function parameters inside the predicate test is UB, we do not attempt to enforce it statically, even bu adding an implicit `const`.
 
 Similarly, given a list of postconditions:
 
 ```c++
double g(int i, int const j)
  [[post r: x(r, i)]]
  [[post q: y(q, j)]];
```
 
a postcondition test sequence is a function defined like this:

```c++
void post_test(double& r, double& q int& i, int const& j) noexcept
{    
  if (PRED(x(r, i))) {} else {
    CONTRACT_VIOLATION_HANDLER();
    std::abort();
  }

  if (PRED(y(q, j))) {} else {
    CONTRACT_VIOLATION_HANDLER();
    std::abort();
  }
}
```

Where:
 
 * The first part of function parameters is constructed by adding lvalue reference to te return type of the function:
   we have as many of these as many different names for the return objects are introduced.
 * The second part of function parameters are references and are constructed from function parameters by adding an lvalue reference.
 * Expressions in precondition predicates are looked up as if they appear in `noexcept` specifications of the functions they appertain to.
 * `PRED(expr)` is either expression `expr` or an expression that returns the same `bool` value as `expr` but has no side effects.

Note: 

 * The return object is always represented by an lvalue reference.
 * Names introduced in postconditions cannot clash with names of function parameters.
 
 
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

There are two slots for performing a precondition test. An implementation must perform the precondition test in at least one of these slots. It can perform the precondition test twice: using both slots.

