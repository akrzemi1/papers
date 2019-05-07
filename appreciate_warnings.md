Appreciate compiler warnings
============================

In this paper we observe that a number of features we are inclined to add to the language whose sole purpose is to cause a potentially buggy program to be ill-formed could be alternatively well handled by compiler warnings. We recomend that instead of adding these features and new keywords, we could shift our focus on clearly communicating where a compiler should issue a warning. We propose it as a guideline for Evolution papers.


Background
----------

First, let's illustrate how this rule has been successfully implemented in some places in C++. Consider a `switch`-statement
where due to missing `break`s the control is spilled from one `case` label to another. The language allows it and there exist
usages of this spillage that make sense, but far more often such spillage is unintended and an indication of a bug. 
Therefore, compilers warn in such cases. But if they warn, they also warn in the cases where te spillage was used conciously
and correctly. In order to avoid false positives C++ has attribute `[[fallthrough]]`. Even though the International Standard has formally nothing to say about warnings issued by implementations, we have an informal comment in the formal document that says:

> [*Note:* The use of a fallthrough statement is intended to suppress a warning that an implementation might
> otherwise issue for a case or default label that is reachable from another case or default label along some
> path of execution. Implementations are encouraged to issue a warning if a fallthrough statement is not
dynamically reachable. *--end note*]

And this is sufficient in practice to communicate the idea to the implementations.

Alternatively, what C++ could have done is to say that a spillage of control from one `case` label to another is a diagnosable
error that renders the program ill formed. But this, apart from detecting bugs in incorrect programs, would also break the
correct programs that use the spillage intentionally in a controlled way. On te other hand, in practice making the spillage
ill-formed has no benefit over compiler warnings. Most compilers provide mode "treat warnings as errors" and allow one to selectively choose which warnings to emit. Even if one is forced to use a compiler without good warnings, it is often an option to use a secondary compiler, with better warning support, only for the purpose of performing a limited static analysis and emitting warnings: one does not even have to link or produce object files from such secondary compilation.

The ability to seletively enabling warnings and then turning warnings into errors is in practice equivalent to generating safer subsets of C++: in the compiler without changes to the International Standard.

`[[fallthrough]]` is just one example. Another is `[[noreturn]]`. Other examples need not involde attributes. It is a common bug to use assignment instead of equality comparison operator inside conditions:

```c++
int i = some_value();
int j = some_other_value();

if (i = j) // BUG: intended `i == j`
  {/*...*/}
```

The majority of compilers warn about such code. But sometimes the assignment inside the condition is intended. For this purpose
Clang and GCC offer the same syntax to disable toe worning quite portably (provided that you port only between GCC and Clang),
by putting redundant parentheses around the expression:

```c++
if ((i = j)) // silence the warning
  {/*...*/}
```


Applicable situations
---------------------

This section lists situations where the language instead of leveraging warnings (and possibly attributes) prefers a hard error,
or where hard error is proposed even though a warning recommendation would do instead. 


### narrowing detection

The authors personally do not appreciate the C++11 feature that narrowing conversions in brace-initialization are treated as errors.
This is because Clang and GCC already detect it under switch `-Wconversion` for any kind of conversion. Forcing it in the language 
with the special treatment of compile-time constants adds no benefits over `-Wconversion` but instead causes lots of troubles: now templates need to be instantiated even in unevaluated contexts in order to determine whether narrowing occurs or not:

```c++
template <int I>
constexpr int get_value() { 
  static_assert(I == -3);
  return 2;
}

auto f() -> decltype(char{get_value<3>()}); // assertion fires
```


Notes
-----


### chaining comparisons

* http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0893r0.html
* http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0893r1.html

### `constinit`

* http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1143r1.md
* http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p1143r0.md

### Herb's `throws`

--------------

* use clang only for warnings (doesn't have to link)

"warning as error" to introduce safer subsets of c++.
