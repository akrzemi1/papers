Author: Andrzej Krzemieński


Contracts: a different take on inexpressible predicates
=======================================================

This paper proposes a solution for handling predicates that cannot be expressed using C++ syntax,
such as "being reachable" for iterators, in preconditions and postconditions, in a different way than
what we have seen in [[P0542r5]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/P0542r5.html).
Rather than annotating preconditions and postconditions as inexpressible, we propose to annotate the
functions used inside as inexpressible.


Motivation
----------

There are predicates common in C++ that are defined formally in human language, but cannot be expressed
as a C++ function, such as that an object of type `const char*` represents a C-style string, i.e.,

1. It is not null (this part is expressible).
2. The array under the pointer is null-terminated.

While there is no way to check this condition at runtime, expressing it using a function signature, 
like `is_null_terminated(str)` is still beneficial as static analyzers can make use of it by treating
it as a symbol, and comparing if one function that requires an argument annotated with this symbol
consumes the value produced by another function that also annotates its return value with the same symbol.
Consider:

```c++
const char* make_name()
  [[post str: str != nullptr && is_null_terminated(str)]]
  ;
  
size_t strlen(const char* str)
  [[pre: str != nullptr && is_null_terminated(str)]]
  ;
  
size_t l = strlen(make_name());
```

Condition `str != nullptr`, which is part of the precondition of function `strlen()`, can be runtime-checked
to assert (partial) program correctness. Regarding the second part, static analyzer can compare symbols:
`is_null_terminated` appears as postcondition of `make_name()` and also as a precondition in `strlen()`,
so assuming that `make_name()` fulfills its contract (this assumption may be verified when compiling `make_name()`),
the analysis can conclude that `is_null_terminated(str)` will be satisfied in the precondition of `strlen`. 
Therefore it is beneficial to have funciton signature `is_null_terminated(const char*)` even if we cannot provide the body
for the function.

Note that a static analyzer can perform a similar analysis for the part `str != nullptr` of the precondition,
however as such static analysis is very resource consuming, it is a reasonable strategy to perform it only
for predicates of which we know that they cannot be runtime checked.

One might think that a trivial solution to this problem is to define function `is_null_terminated()` as always returning `true`:

```c++
constexpr bool is_null_terminated(const char* str)
{
  return true;
}
```

This way we (1) have a signature and (2) do not rise false alarms when this function is "evaluated" at runtime. 
(Of course no compiler will generate code for evaluating such trivial function.)

But this will not solve the problem, because static analyzer does not know that it should treat this function
only as a symbol: it will look inside, see `return true;` and conclude that this predicate is always satisfied,
regardless of the argument value.


Previous solution
=================

The solution to this problem in [[P0542r5]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/P0542r5.html) was to
annotate the precondition or postcondition as special: designated to be treated as symbol and subject to static analysis.
Because such annotation appertains to the entire predicate, if some part of a predicate can be runtime-checked and the other
cannot, the predicate needs to be split across two declartions:

```c++
const char* make_name()
  [[post str: str != nullptr]]
  [[post axiom str: is_null_terminated(str)]]
  ;
  
size_t strlen(const char* str)
  [[pre: str != nullptr]]
  [[pre axiom: is_null_terminated(str)]]
  ;
```

Word `axiom` is used to indicate that this precondition needs to be verified by the static analyzer rather than
runtime-checked.

This solution had some issues. First, it conflated the notion of "contract level" (which says about the relative runtime cost
of evaluating the function body versus evaluating the predicate) with the property of requiring and not requiring the function
to be ORD-used (i.e., requiring that the function should have a definition). This side-tracked people into thinking what would be the meaning af an `axiom`-level precondition with a perfectly expressible and cheap-to-evaluate predicate, and if it can be
reused for some other purposes.

Second, the choice of the word mislead many experts to believe that this annotation has semantics similar to Clang's
`__builtin_assume()`, which is used to empower the compiler to do potentially dangerous code transformations that may
change the semantics of a program.

Third, it sometimes requires a precondition that is composed of expressible and inexpressible parts to be separated into
two annotations. If the two parts are connected by a logical conjunction (operator `&&`) this is easily doable, as in the above
example. However, if operator `||` is used, as in:

```c++
size_t consume_optional_name(const char* name)
  [[pre: name == nullptr || is_null_terminated(name)]]
  ;
```

It is not possible, and the part that is easily runtime-checked must now be performed by the static analyzer, wasting its
precious resources.


This proposal
-------------

The alternative that we propose is that it is the functions used in predicates  that should be indicated as subject
to static analysis, rather than preconditions and postconditions. 

In this proposal the examples above would be written as:

```c++
constexpr bool is_null_terminated(const char* str) [[symbolic]]
{
  return true;
}

const char* make_name()
  [[post str: str != nullptr && is_null_terminated(str)]]
  ;
  
size_t strlen(const char* str)
  [[pre: str != nullptr && is_null_terminated(str)]]
  ;
  
size_t consume_optional_name(const char* name)
  [[pre: name == nullptr || is_null_terminated(name)]]
  ;
```

A new attribute spelled `[[symbolic]]` (the chocie of name is negotiable, of course) can only appertain to functions. It indicates that the condition indicated by the function cannot be expressed as a C++ predicate. It is information to the
static analyzer that the function, wherever it appears, should be treated as a symbol and its body should not be made use of
during static analysis. 

The body of a funcion annotated with `[[symbolic]]` is only needed when evaluating the function at runtime. Such functions
have special meaning only to the static analyzer. The compiler still sees it as a normal function and generats the code that 
will be potentially runtime-evaluated. In case of function `is_null_terminated()` the implementation is trivial. We return `true`, because we do not want to signal a contract violation in the situation where we do not know for sure that the contract is violated. So the rule is, "unless you are sure that the contract is violated, return `true`". 

However, in case of other predicates a function can check expressible subsets of an inexpressible predicate. Consider the implementation of `is_reachable()`:

```c++
template <input_iterator It, sentinel_for<It> S>
bool is_reachable(It begin, S end)
  [[symbolic]] // not expressible
{
  return true; // don't know the answer, return safe answer
}

template <typename T>
bool is_reachable(T* begin, T* end)
  [[symbolic]] // still not expressible as a whole
{
  if (std::less<T*>{}(end, begin)) // but has expressile subset
    return false; // not reachable for sure
  
  return true; // maybe reachable, maybe not reachable
}
```

More, in case we have a special debugging-purpose iterator, it may have enough redundant information to tell if it is reachable from another iterator (like pointer to the undelying container and an index in that container). In that case we can provide an overload that is cheap to evaluate at runtime, and we will *not* mark it as `[[symbolic]]`:

```c++
template <typename T>
bool is_reachable(vector_debug_iterator<T> begin, vector_debug_iterator<T> end)
  // no attribute: it is perfectly expressible
{
  return end.is_reachable_from(begin);
}
```

Now, a function that uses the requirement `is_reachable()` does not need to know about all these overloads. It just uses the name:

```c++
template <input_iterator It, sentinel_for<It> S, invokable<value_type<It> F>>
void for_each(It begin, S end, F f)
  [[pre: is_reachable(begin, end)]]
;
```

Now, the decision to verify the precondition symbolically or by run-time checks depends on a particular specialization of `is_reachable()`. But the user of `for_each()` is not distracted with the information of whether this will be verified symbolically or at run-time. 

Thus, the property of being inexpressible (or being intended for static analysis) is not conflated with the notion of "levels".
The only levels we have are `default` and `audit`. Also, the precondition expressed as:

```c++
[[pre: is_reachable(begin, end)]]
```

Does not send any confusing message like "treat me as `__bltin_unreachable()`".

In order to prevent suspicious/unintended usages of `[[symbolic]]` functions we additionally require that they can only be invoked inside either precondition/postcondition annotations or other `[[symbolic]]` functions. Maybe, we could additionally require that `[[symbolic]]` functions can only have return type `bool`, but it is not clear at this point if this requirement is needed. 


Acknowledgements
----------------

Gor Nishanov has reviewed this proposal and offered a valuable feedback.


References
----------

[[P0542r5]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/P0542r5.html) -- G. Dos Reis, J. D. Garcia, J. Lakos, A. Meredith, N. Myers, B. Stroustrup, "Support for contract based programming in C++".
