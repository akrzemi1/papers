Appreciate compiler warnings
============================

In this paper we observe that a number of features we are inclined to add to the language whose sole purpose is to cause a potentially buggy program to be ill-formed could be alternatively well handled by compiler warnings. We recommend that instead of adding these features and new keywords, we could shift our focus on clearly communicating where a compiler should issue a warning. We propose it as a guideline for Evolution papers. We also question the common conviction that the International Standard is not able to mandate compiler warnings.


Background
----------

First, let's illustrate how the rule we propose has been successfully implemented in some places in C++. Consider a `switch`-statement
where due to missing `break`s the control is spilled from one `case` label to another. The language allows it and there exist
usages of this spillage that make sense, but far more often such spillage is unintended and an indication of a bug. 
Therefore, compilers warn in such cases. But if they warn, they also warn in the cases where te spillage was used conciously
and correctly. In order to avoid false positives C++ has attribute `[[fallthrough]]`. Even though there is no requirement that implementations should issue a warning, we have an informal comment in the formal document that says:

> [*Note:* The use of a fallthrough statement is intended to suppress a warning that an implementation might
> otherwise issue for a `case` or `default` label that is reachable from another `case` or `default` label along some
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

If detecting narrowing was the responsibility of the compiler vendors, they would decide to what extent they want to engage in detecting narrowing, and if it would require the instantiation of class templates and bodies of function templates inside unevaluated operands, they would just provide a different definition of "narrowing".


### Chaining comparisons

Regarding the proposal for chaining comparisons ([[P0893r1]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0893r1.html)), the motivation for the paper is twofold:

1. Conveniently express relations between three or more values: `begin <= it <= end`.
2. Detect bugs where currently the comparisons are written down this way even though the semantics are different.

The first motivation is probably too weak to warrant a change; it also has significant cost: it does not map onto the expression tree model of C++. The second motivation is stronger: we do want to detect bugs in the code. However, this can be addressed with a compiler warning. Compilers today can detect this type of a bug. In fact GCC already gives the warning under switch `-Wparentheses`. An alternative to [[P0893r1]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0893r1.html) is, instead of a language feature that renders such operator chaining ill-formed, to encourage the implementations to emit a warning in this case. Such encouragement can say that if comparisons are chained (with no parentheses) the implementation is required to emit a diagnostic message.


### `constinit`

The initial `constinit` proposal ([[P1143r0]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p1143r0.md)) proposed an addition of atribute `[[constinit]]` that would render the program ill-formed it constant initialization cannot be performed. The solution with an attribute was rejected because a presence or absence of an attribute cannot change a program from well-formed to ill-formed or vice versa. This is correct; therefore the next revision ([[P1143r1]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1143r1.md)) proposed a new keyword instead of an attribute, which added to the list of keywords with prefix "const": `const`, `const_cast`, `constexpr`, `consteval`, `constinit`, which adds confusion for the programmers: which of the last three am I supposed to use when?

However, there was a different way to fix the original proposal: leave the attribute and rather than treating the program as ill-formed require the implementations to emit a diagnostic message. The wording would look like this:

> If a variable declared with the `[[constinit]]` attribute has dynamic initialization ([basic.start.dynamic]),
> the implementation shall emit a diagnostic message.


### `try`-expressions in P0709

[[P0709r2]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0709r2.pdf) (Zero-overhead deterministic exceptions: Throwing values) in section 4.5.1 proposes `try`-expressions that could be used to explicitly mark exceptional paths in te code.
Later, in section 4.5.2 it proposes that the program should be ill-formed if an exceptional path in functions declared wit the new `throws` specifier is not marked with the `try`-expression:

```c++
int f1() throws; // throws in a new, better way
int f2() throws;

int main() {
  return f1() + f2();     // error, f1() and f2() could throw
  return try f1() + f2(); // ok, covers both f1() and f2()
}
```

The idea is that the enforcement would only be applicable in the newly added functions declared with `throws`,
which obviously do not exist in the current code bases; therefore existing code bases would be unaffected by this requirement. 
The change would be backwards-compatible. However, given that many functions will potentially throw due to free store allocation failures (counter to what section 4.3 implies -- this is explained in paper [[P1404r0]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1404r0.md)) the amount of `try`-expressions required to make the program compile
may become overwhelming. It is better if the enforcement of explicitly annotating exceptional paths with `try`-expressions is an opt-in feature. This, again, can be achieved with compiler warnings. The International Standard could require of the implementations to produce one diagnostic message wherever an exceptional path is not annotated with a `try`-expressions in a `thorws`-annotated function, and anoter diagnostic message wherever an exceptional path is not annotated with a `try`-expressions in old-style functions. The implementations would then have freedom for providing a switch to suppress some of the required diagnostic messages.


References
----------

[[P0893r1]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0893r1.html) Barry Revzin, Herb Sutter, "Chaining Comparisons".

[[P1143r0]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p1143r0.md) Eric Fiselier, "Adding the `[[constinit]]` attribute".

[[P1143r1]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1143r1.md) Eric Fiselier, "Adding the `constinit` keyword".

[[P0709r2]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0709r2.pdf) Herb Sutter, "Zero-overhead deterministic exceptions: Throwing values".

[[P1404r0]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1404r0.md)) Andrzej Krzemieński, "`bad_alloc` is not out-of-memory!".
