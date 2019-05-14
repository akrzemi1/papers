Impure expressions in Contract Conditions
=========================================

The specification of contract statements in the current [[WD]][1] is that any side effect that occurs during the evaluation of
any contract condition is undefined behavior. The rationale for this is that programmers are expected to use only *pure* 
(referentially transparent) expressions in contract conditions, as otherwise reasoning about these conditions as predicates is 
impossible. 

This does not seem to take into account that there exist functions that are very good candidates for predicates but that have
"benign" side effects (such as doing logging on the side). In this paper we explore this problem in detail, and propose to allow 
side effects in contract conditions and specify what happens with them in different modes.  


Background
----------

The [[WD]][1] when referring to preconditions and postconditions uses term, "to *hold*", which only makes sense for *predicates*
in the mathematical sense. In programming, this corresponds to the notion of a *pure*, or *referentially transparent* function. Such functions/expressions have two essential properties:

1. No side effects.
2. They are deterministic: each time they are invoked with the same values of inputs they render the same value: they do not depend on any state other than the state of the inputs.

These two features combined mean that the evaluation of a referentially transparrent expression can be substituted with the value it produces, without changing the program behavior. And vice versa: a value can be replaced at any point with a referentially transparent expression that produces this value, without changing the behavior of the program.

In C++ there is no way to statically detect if a given expression is pure. So, one way of dealing with it is to say that the 
compiler can proceed as if contract conditions were pure, and the programmers are forced to make sure that they are, with no help from compiler. This is equivalent to saying that side effects or non-determinism in contract conditions is UB.

In [[WD]][1] side effects in contract conditions are UB, but non-determinism is not. Therefore, the following precondition causes no UB and its behavior is well defined according to the specification of the abstract machine:

```c++
int f(int i)
  [[expects: get_local_time().hour_24() < 12]];
```

The expression will be evaluated at indicated points when checking mode is enabled. And depending on whether the moment of evaluation occurs before or after the noon the preconditoin "will hold" or not. If checking mode is disabled the expression 
may be evaluated or not (unspecified), and if it is after noon the behavior is undefined.

------------------------------

Positive sides of UB
--------------------

Can a predicate be executed more than once? -- UB allows this.


References
----------

[1]: http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/n4810.pdf
[[WD]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/n4810.pdf) -- Richard Smith, N4800, "Working Draft, Standard for Programming Language C++".

[2]: http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0380r0.pdf
[[P0380r0]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0380r0.pdf) -- G. Dos Reis, J. D. Garcia, J. Lakos, A. Meredith, N. Myers, B. Stroustrup, "A Contract Design".
