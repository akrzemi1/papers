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

Thus, we are facing a technical trade-off here between how much we can express and the simplicity of the model.
