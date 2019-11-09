Proposed nomnclature for contract-related proposals
===================================================

Ths document proposes a number of definitions that we recomend for authors of contract-related proposals and SG21 members to use, in order to avoid misunderstandings.

One sub-goal was to avoid, whenevr possible, terms:

* "assume" -- as it is not clear who assumes, is it a voluntary action or a command, and what the implication of the assumption is. Or it could mean "function *assumes* that its precondition holds".
* "expect" -- as it is not clear who expects and why would anyone else care what the other person or function expects; you 
  can "expect" that people will use your function incorrectly. 
* "assert" -- as "assert" could mean "trust me, I say it and this is true", or "test this hypothesis for me".
* "check" -- as it implies that some run-time checking will be performed or required, whereas some predicates are not even 
  expressible as code that could be executed.


Recommended definitions
-----------------------

#### *Contract annotation*

It has a *location* in code: just before the function execution begins, or just after the function execution end successfully.
It has a *predicate*, such as `p != nullptr`. Its "semantics" is: if the execution reaches the contract anntation,
and the predicate can be determined (not necessarily by evaluating it) to be `false`, then the program has a *contract violation*.
The information about a contract violation in the program
can be used in different ways by different tools, including the compiler.  

Thus, contract annotation helps provide a definition of a bug that is understandabe by machines. Becuse contract violation is for sure a manifestation of a bug.


#### *Contract violation*

A contract violation results from an interaction between the code before the contract annotation and the contact annotation 
itself. The behavior of a program with a contract violation can differ from one translation to the other based on compiler 
switches. This is similar to implementation-defined behavior except that we try to specify in more detail what the variance
in behavior is, and under what conditions.

A program with a contract violation can still be a well-formed, UB-free program. Unless other UB kicks in later on, you can
still reason about the behavior of the program in terms of operations on the abstract machine.

Contract violation is a symptom of a bug located either before the contract annotation or inside its predicate. 
Contract annotations can detect symptoms of bugs, but not bugs themselves. 


#### *Fact*

An information that an implementation has about state of variables at the given point of program executon.
A fact can be used for different things by the implementation, including code transformtions. E.g.,

```c++
int normalize(int i)
{
  if (i > 0) return 1;
  if (i < 0) return -1;
  
  // at this point it is a fact that `i == 0`
  return i; // can be replaced with `return 0;`
}
```

#### *Derived fact*

A fact, like the one illustrated above, that the implementation obtains by means other than provision that the International Standard places no requirements on the implementation when program hits an undefined behavior.


#### *Injected fact*

A fact obtained by provision that the International Standard places no requirements on the implementation when program hits an undefined behavior. E.g.,

```c++
int f(int i)
{
  __builtin_assume(i == 0); // UB if i !== 0
  
  // at this point it is an (injected) fact that `i == 0`
  return i; // can be replaced with `return 0;`
}

int g(int i)
{
  if (i != 0)
    *((void*)0) = 0; // UB
  
  // at this point it is an (injected) fact that `i == 0`
  return i; // can be replaced with `return 0;`
}
```


#### *Injected run-time check*

From any contract annotation in location *L* with predicate `P`, an implementation can under unspecified conditions (but we try to specify it with "levels", "build modes" and other "toggles") insert at location *L* a piece of code that is executed at run-time, possibly with side effects, of the form:

```c++
{
  if (!(P)) HANDLE_CONTRACT_VIOLATION();
}
```

This is called an *injected run-time check*. `HANDLE_CONTRACT_VIOLATION()` is subject to further implementation-defined behavior, such as calling usr-provided funcion, or doing nothng, or maybe doing something even different.

This definition leaves open the question, whether `P` should be allowed to have side effects or not, as well as what "side effects" would mean in this context.

