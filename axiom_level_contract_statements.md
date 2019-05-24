Paper number: XXXXXX <br>
Date: XXXX-XX-XX <br>
Author: XXXXX <br>
Email: akrzemi1 (at) gmail (dot) com <br>
Audience: EWG, CWG


Axiom-Level Contract Statements
===============================

Current [[WD]][1] allows axiom-level contract statements to be evaluated at runtime. This is contradictory with the design
goals for axiom-level contract statements outlined in [[P0380r0]][2], which explicitly notes that the goal for axiom-level 
contract statements is to allow functions without definitions and possibly with side effects. In this paper we propose to make it clear that a program is 
guaranteed, in any build level, to compile and link when conditions in axiom-level contract statements contain references
to objects and functions without definitions.


### Non-goals

At the same time we want to preserve the guarantee from [[WD]][1] that if the implementaiton can somehow determine the value that the 
predicate would return, it should be able to use this information for optimization or correctness verification purposes. 
Unconditionally having such optimizations are controversial and considered a defficiency in [[WD]][1] by some parties, but we consider it an orthogonal issue
to the one discussed in this paper.

Note that by not allowing axiom-level contracts to be evaluated we prevent them from ever having side effects (a runtime property of the evaluation of a predicate).
This implicitly allows the use of predicates in axioms that would otherwise have side effects if used in a checked contract level.


What we need
------------

We need to be able to declare predicates like:

```c++
template <InputIterator I, Sentinel<I> S>
  bool is_reachable(I b, S e); // never defined

template <typename T>
  bool points_to_heap_object(T* p); // never defined
```

They are unimplementable, therefore they are only declared but never defined. They are still useful for static analysis. We want to be able to put them in axiom-level contract statements and have the guarantee that we will never get a linker error saying 
that an odr-used symbol is missing, regrdless of any build level.

At the same time, it is plausible that the implementation can understand the semantics of our condition and by some other means 
it can compute the result without causing any side effects and without odr-using any new entity in the condition. Such result could be used for optimization purposes, or for additional correctness validation. The current [[WD]][1] allows this and we do not want to prevent this.

Note that with this change contract statements with level `axiom` will become substantially different from other levels: only for them the "no linkage problems" guarantee applies. (`audit` and `default` levels require defineitions even if they are not evaluated in the current build level.  For those levels we explicitly prefer retaining that ODR use to minimize the risk of writing code that links with some build levels and fails to link with others, which is a significant benefit contracts can have as a language facility over being implemented solely with macros.)

To summarize our goal:

1. Missing definitions of entities (objects or functions) in the condition of an axiom-level contract statement never make the program ill-formed. Same as inside `sizeof()` or `decltype()`.
2. Optimizations still potentially enabled if there is a way to determine the predicate result without violating point 1.


Proposed wording
----------------

Modifications apply to section [dcl.attr.contract.cond].


#### Modify paragraph 4:

During constant expression evaluation, only predicates of checked contracts are evaluated. In other contexts, it is unspecified whether
the predicate for a <ins>`default` or `audit` level contract</ins> that is not checked under the current build level is 
evaluated; if the predicate of such a contract would evaluate to `false`, the behavior is undefined.

#### Add paragraph at the end:

The predicate `p` of a contract condition with `axiom` *contract-level* is an unevaluated operand.  If the predicate of such a contract
would evaluate to false, the behavior is undefined.

[*Example:*
```c++
bool pred(int i); // never defined
void g(int *p)
{
  [[ assert : p && pred(*p) ]];
  if (p != nullptr)  // this check can be elided, g will always dereference p.
    *p = 0;
}}
```
Implementation can eliminate the check `p != nullptr` in function `g`. If `p` was null, the assertion in `g` would evaluate
to `false`, which would be undefined behavior. The potential contract violation can be determined without the call to function
`pred` owing to the semantics of operator `&&`. 
*--end example*]

#### Alternate wordings for this paragraph:

The "If the predicate of such a contract would evaluate to false" wording is terse and potentially a source of confusion.  Alternate
wordings to consider that we believe would have the same meaning might be one of the following:

  If an implementation is able to determine, by means not specified by this International Standard, what value would be returned by evaluted `p`,
  and this value is `false`, ...

  If an implementation is able to evaluate an alternate predicate with no side effects that returns false only if the contract predicate returns false,
  and such an alternate predicate would return false, ...


Acknowledgements
----------------

Many people in EWG reflector helped shape this proposal, in particular Joshua Berne and Tony Van Eerd.


References
----------

[1]: http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/n4810.pdf
[[WD]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/n4810.pdf) -- Richard Smith, N4800, "Working Draft, Standard for Programming Language C++".

[2]: http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0380r0.pdf
[[P0380r0]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0380r0.pdf) -- G. Dos Reis, J. D. Garcia, J. Lakos, A. Meredith, N. Myers, B. Stroustrup, "A Contract Design".

[3]: http://www.open-std.org/JTC1/sc22/wg21/docs/papers/2019/p1429r1.pdf
[[P1429r1]](http://www.open-std.org/JTC1/sc22/wg21/docs/papers/2019/p1429r1.pdf) -- Joshua Berne, John Lakos, "Contracts That Work".
