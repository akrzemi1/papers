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
compiler can proceed as if contract conditions were pure, and the programmers are forced to make sure that they are, with no help
from compiler. This is equivalent to saying that side effects or non-determinism in contract conditions is UB.

In [[WD]][1] side effects in contract conditions are UB, but non-determinism is not. Therefore, the following precondition causes no UB and its behavior is well defined according to the specification of the abstract machine:

```c++
int f(int i)
  [[expects: get_local_time().hour_24() < 12]];
```

The expression will be evaluated at indicated points when checking mode is enabled. And depending on whether the moment of evaluation occurs before or after noon the preconditoin "will hold" or not. If checking mode is disabled the expression 
may be evaluated or not (unspecified), and if it is after noon the behavior is undefined.


Positive sides of UB
--------------------

Transformations based on pure expressions can be very useful: 

* we can skip the evaluation of a checked precondition if we know by other means what the result will be;
* we could evaluate the checked precondition at a different place than required by the language;
* we could evaluate the precondition even if it is not allowed at a given point by the standard.

When we specify impurity as UB we can still apply all these transformations. For impure functions these transformations will obviously change the semantics of the program, but it fits into the provision of UB: anything can happen when the programmer didn't do his part. Thus, for pure expressions these transformations do not change the program semantics, and for impure ones, you have UB, so you cannot complain about anything.

The only problem is, how can a programmer be sure that all his contract conditions are pure?


Negative sides of UB
--------------------

The problem with UB is that it is very permissive. It allows the compiler to do all the useful transformations of the program listed above, but it also, as part of the same package, allows implementations to do other things, which can have unintended consequences. For instance, the following function `fun()` has a side effect in its precondition:

```c++
int half(int i)
{
  log("half() invoked");
  return i / 2;
}

int fun(int i)
  [[expects: half(i) < 128]];
```

Having a side effect in a precondition that is invoked is a UB. Compiler can assume that UB never happens. Therefore compiler can assume that function `fun()` is never invoked in a program compiled in build mode that checks default-level contract conditions. Therefore it can transform the following code:

```c++
void test(int x)
{
  if (x > 0)
    fun(x);
  else
    gun(x);
}
```

into:


```c++
void test(int x)
{
  gun(x);
}
```

It can further assume that `x <= 0`, even though this assumption was probably never intended by the programmer. 

And the scarry part is, the author of `fun()` may not see the body of function `half()` and may have no chances of knowing that
even though it is deterministic, it is doing some side effect not observable in and irrelevant to our context.

People have expressed their concerns about compiler being able to optimize based on potentially unsatisfied preconditions, which
causes an unpredictable change in semantics when functions are called out of contract (contract based optimizations). But in those cases at least it is made clear ot the programmers (who do mistakes) when this will happen: they have a chance to call functions within contract. In contrast to this, in the problem described in this paper, programmer does not even know if calling a given function will cause, however benign, side effects.


------------------------------



Can a predicate be executed more than once? -- UB allows this.


References
----------

[1]: http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/n4810.pdf
[[WD]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/n4810.pdf) -- Richard Smith, N4800, "Working Draft, Standard for Programming Language C++".

[2]: http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0380r0.pdf
[[P0380r0]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2016/p0380r0.pdf) -- G. Dos Reis, J. D. Garcia, J. Lakos, A. Meredith, N. Myers, B. Stroustrup, "A Contract Design".
