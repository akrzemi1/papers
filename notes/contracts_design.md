Contracts Design
================

This paper describes the design goals and design criteria for the feature that we (perhaps inacurately) call "contract support".  

### Contracts Wish List

Because we use terms "contract support", "C++ contracts", "Contracts Study Group", programmers may develop expectations that the feature we want to build will be able to reflect *any* part of the function contract. For instance:

  * that the complexity of a function is linear,
  * that the function offers a commit-or-rolback exception safety garantee.

However, our goal is not to express any part of the function contrct: only a subset of it. In fact, a small subset. Regarding the two above examples, hardly anyone will protest that they are left behind, but we do not even pretend to be able to express all things that we consider "postcondition". For instance:

```c++
void foreach(Range auto range, Function auto fun);
```

The function ensures that `fun` has been called exactly `size(range)` times. We do not aspire to provide support for such gurantees. Instead our goal is to provide something that is:

  * Simple to learn, use, implement;
  * Covers a reasonably vast portion of "function contract".

Thus, we are facing a design trade-off here between how much we can express and the simplicity of the model.



The High-Level model
--------------------

The mmodel that we propose, which has a long history in Computer Science, is that a programmer is able to declare in the code
*correctness annotations* which consist of two parts:

  1. The predicate *P* on the program state (more like in the mathematical or functional language sense).
  2. The location *L* in the program flow. 

The meaning of such declaration is that if *P* inspecting the program state is evaluated at the indicated location *L*, and the result is `false`,
then we can be sure that there is a bug in the program logic: either before or exactly at *L* (including inside the *P*). This information could be used in different ways by different tools.

The locations that the programmer can indicate are:

  1. Just before a given function is invoked. This is the precondition-annotation.
  2. Just after the function exits normally (no exception, no `longjmp`). This is the the postcondition-annotation.
  3. Exactly where a given statement is executed. We could call it a statement-annotation for uniformity, but we mean the thing that we usually call "assertion".

We call them "annotations" because they do not change the maning of the program. They are used to "classify" the execution paths. 
These annotations can be used to instrument the code, but this is just one of the possible outcomes: not guaranteed. 
A different use case can be to conduct correctness roofs.

The presence of correctness annotations does not make the program more correct; nor does the removal of the contract annotations from the program make it less correct. (Here, we measure the "correctness" by the number of unintended constructs in the program source code; that is, programmer bugs.) The runtime checks enabled by these annotations only make the bugs manifest in a different way.

We have two expectations of the predicates: they are contradictory to some extent. We want them to be:

  1. Reflected as a C++ expression returning `bool`, so that they do not have to live in comments, and could be type-checked.
  2. Like mathematical predicates, so that humans can think of them as properties or adjectives, and can easily understand 
     the contract by reading the function declaration. (Unfortunately, this may be the first time that a programmer is exposed to the function's contract.)

The two goals can be achieved for sufficiently simple expressions and sufficiently simple types. For instance `i != 0`, where `i` is of type `int`, satisfies both criteria. But `i + j < k`, where `i`, `j`, `k` are of type `int`, cannot be safely said to satisfy them, because it is undefined behavior if evaluated for some values of `i` and `j`. Similarly, `is_fine(i)` may not satisfy both criteria, because the function may have a side effect visible to other parts of the program. 

There is no way to prevent statically exprssions from being side-effect-free. So, the way we approah this is to allow the compiler to perform runtime checks based on the 
correctness annotations and to apply transofmations on them as if they were side-effect free (even if they are not). One of the goals of this is to discourage the programmers from putting any code there that cannot be easily expressed as a property or adjectve.  

