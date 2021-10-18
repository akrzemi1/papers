Execution model for pre- and postconditions
===========================================

There is a certain dychotomy when we describe preconditions and postconditions. They come from functional languages where terms "value" and "predicate" have intuitive and mathematically sound meaning. In contrast, in an imperative language like C++ we have objects, object states and functions (potentially with side effects). We believe that a lot of difficulties in designing and communicating the contract support in C++ stems from this failure to differentiate between the notions from functional and imperative paradigms.

In this paper we want to present the execution model for pre- and postconditions in a way tat does not confuse the two paradigms. We want the programmers to be able to reason in terms of "value" and "predicate" as much as it is feasible, but we want the formal description of the mechanics in  *Eval_and_abort* mode to be described in terms "object" and "function".


The model
---------

We choose the following model for preconditions and postconditions.
The programmer can declare *contract annotations* on function declarations.
These contain expressions of type `bool` that will be evaluated at well 
defined points in the program (just before the function is executed, 
just after the function returns). When any such expression is evaluated, 
it can observe the state of the program at this point in time.
However, because they appear in function declarations, there is no caller
in scope, nor is the function body. Thus, the state of the program can only
be observed either through global-like objects (namespace-scope objects,
static class members) or function parameters. Reference function parameters
give access to objects from the caller's scope.

If any such expression in a contract annotation evaluates to `false` this indicates
that the program state is *incorrect*. If it evaluates to `true` no assertion about the program state is made.

This model is 'simple': easy to explain, easy to implement. However, it also has limitations:

 * it cannot express all things that we would intuitively want to express, and
 * it sometimes behaves against the intuition from the functional programming world.


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



Pre/postcondition tests are not assertions
------------------------------------------

This section illustrates that the semantics of pre- and postcondition checks
cannot be expressed in terms of assertions put manually in the caller code or inside the function body.

In the following examples we will use the following class template for demonstration.

```c++
template <typename T>
struct wrap
{
  wrap(T const& v);      // converting constructor
  wrap(wrap&&) = delete; // not movable
  bool ok() const;
};
```

Consider the following function definition,

```c++
wrap<int> fun(wrap<int> a)
  [[pre: a.ok()]]
  [[post r: r.ok()]]
{
  char j = 1;
  return j;
}
```

and the following function call.

```c++
int i = 1;
// loc A
fun(fun(i));
```

In the caller code, we cannot put an assertion at location A 
that would correspond to the precondition in function `fun`.
This is because the type of function argument is of type `wrap<int>`
and there is no object of this type in the caller's scope.
The second issue might be the access checking: the caller might not have
the sufficient access. The third issue is the overload resolution. 
In this proposal names are looked up in the scope of the function declaration:
not in the caller's scope.

Similarly, an assertion appearing as the first instruction in the
function body is not equivalent to the precondition check because of  
the overload resolution, which behaves differently inside the function body.

The postcondition test of the function `fun` cannot be expressed as an assertion 
anywhere in the function body, because there is no way to put it. Because we are 
inspecting the return object, this would have to happen after the return object has been
initialized. That would have to be after the `return` statement, but then it is too late;
and more importantly, the return object has no name in the function body. We cannot first
create it, then test the postcondition and then return it, because that would require
our type to be movable, but it is not.

We cannot also express the postcondition test of the first funciton `fun` as an assertion in the caller,
because there is no named object of type `wrap<char>` visible in the caller. We could not split
the expression `fun(fun(i))` into two intructions and introduce a named variable, because then
we would need to move it, and this would not work for a movable type.


Function parameters
-------------------


The lifetilme of an object represented by a non-reference function parameter is peculiar: 


 1. It is initialized by the caller.
 2. It is inspected and modified by the callee.
 3. It is destroyed by the caller.
 
(One can confirm that by using a `noexcept` function and a parameter type that throws from its
constructor, any modifying member function and the destructor.)

Therefore, when a precondition annotation refers to a non-reference function parameter,
the corresponding precondition test will observe the value of the object immediately
after its initialization. Whereas, when a postcondition refers to the same parameter,
the corrsponding postcondition test will observe the value of the same object immediately
before its destruction, potentially after it has been modified inside the function body.
Consider:

```c++
void f(int i)
  [[pre:  i == 1]]
  [[post: i == 2]];
```

When we read the above declaration using the intuition from functional programming,
it may appear illogical: in mathematics `i` is a value, and this property is not related
to the passage of time. In contrast, in C++, `i` is an object whose state may change in
time, and the above contract annotations inspect the state at two diffrent points in time:
immediately after initialization and immediately before destruction. What use can the 
caller make of it? 

Regarding the precondition, it is the caller who initialized the value, so the caller
can make sense of the value. In the postcondition, we have two cases:

 1. The function did not modify the object. In this case, the caller can make 
    sense of the object state, because it is the same state that the caller initialized.
    
 2. The function modified the object. In this case the caller has no clue what the
    state of the object is, and can make no sense of it.

Now, consider this more practical example:


```c++
int select(int lo, int hi)
  [[pre: lo <= hi]]
  [[post r: lo <= r && r <= hi]];
```

The postcondition has a clear intuitive meaning: the return value must fall 
between the values that `lo` and `hi` had upon initialization. A static analyzer
performing type-and-effect analysis will also interpret it this way. However,
a postcondition test in our model has no way of inspecting the past states of function
parameters: it can only observe the state of the parameters after the function has finished;
unless, of course, we can guarantee that the fnction does not modify the inspected function
parameters.

This required guarantee that the function does not modify its non-reference parameters
can be approached in two ways:

 1. Trust the function authors that they understand our model and that they will
    never try to modify function arguments.
 2. Enforce it statically: make the program ill-formed when a function non-reference parameter
    is referenced in the postcondition and it is not declared `const`.
    
We believe that it is not possible for the programmers to exercise this level of discipline.
First, it is a common programming practice to modify function parameters inside function body.
A number of STL implementations do this:

```c++
auto for_each(auto first, auto last, auto f)
{
  for (; first != last; ++first)
    f(*first);
  return f;
}
```

Second, the contract annotaiton appears in the function declaration and the function body may
be in a separate translation unit authored by a separate person, who may not even know that 
a contract annotation has been added. We have the feature of implicit move in `return`-statement,
which may modify a function parameter, without any visible indication: the programmer may not
be even aware that the function parameter is moved from (and hence modified) before being inspected. 
     
