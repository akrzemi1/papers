Execution model for pre- and postconditions
===========================================

There is a certain dychotomy when we describe preconditions and postconditions. They come from functional languages where terms "value" and "predicate" have intuitive and mathematically sound meaning. In contrast, in an imperative language like C++ we have objects, object states and functions (potentially with side effects). We believe that a lot of difficulties in designing and communicating the contract support in C++ stems from this failure to differentiate between the notions from functional and imperative paradigms.

In this paper we want to present the execution model for pre- and postconditions in a way tat does not confuse the two paradigms. We want the programmers to be able to reason in terms of "value" and "predicate" as much as it is feasible, but we want the formal description of the mechanics in  *Eval_and_abort* mode to be described in terms "object" and "function".


The model
---------

We choose the following model for preconditions and postconditions. We give te programmer a tool for defining expressions of type `bool` that will be evaluated at well defined points in the program (just before the function is executed, just after the function returns). When any such expression is evaluated, it can observe the state of the program at this point in time -- global variables (namespace scope, class `static` members), function parameters -- and based on this return `false` if the state is incorrect (`true` means we do not know if the program is correct or not). This model is simple: easy to explain, easy to implement. However, it also has limitations: it cannot express all things that we would intuitively want to express, and it sometimes behaves against the intuition from the functional programming world.


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
  3. start function call (`noexcept` barrier)
  4. **perform precondition test sequence** (inspecting function argument objects)
  5. run function body
  6. initialize the return object
  7. call destructors of automatic objects
  8. **perform postcondition test sequence** (inspecting function argument objects, and return object)
  9. end function call (`noexcept` barrier)
 10. **perform postcondition test sequence** (inspecting function argument objects, and return object) 
 11. call destructors of function parameters

There are two slots for performing a precondition test. An implementation must perform the precondition test in at least one of these slots. It can perform the precondition test twice: using both slots.

The difference between performing the precondition/postcondition test inside and outside the function boundary is not observable in the MVP, because it has been stripped off the features where that difference would be observable. There is no logging, and the entire precondition/postcondition test is `noexcept`. This may change as extensions are added in the future:

 1. If we allow logging from violation handlers, tools that can reflect on the source location, 
    such as macro `__LINE__` or `std::source_location::current()` can behave differently inside or outside the function.
 2. If we allow the violation handlers that throw exceptions, then invoking the violation handler for contract
    annotations of `noexcept` functions may end in `std::terminate()` or not, depending on whether the test is performed inside or outside of the function.


### Wrong intuition

In the following examples we will use the following class template for demonstration.

```c++
template <typename T>
struct wrap
{
  wrap(T const& v);      // converting constructor
  wrap(wrap&&) = delete; // not movable
  bool is_fine() const;
};
```

```c++
wrap<int> fun(wrap<int> a)
  [[pre: a.is_fine()]]
  [[post r: r.is_fine()]]
{
  return 1;
}
```

```c++
int i = 1;
wrap<wrap<int>> s = fun(i);
```

