Decisions
---------

* Function pointes/object types should include the precondition? (no)
* Covariant and contravariant pre/post conditions in overriding functions?
* accessing private members in contracts on member functions?
* UB on side effects in conditions?

Side effects in conditions
--------------------------

* Compiler allowed to evaluate a checked contract 0, 1, or more times.
* Compiler allowed to remove one of two indentical condition evaluations if they are consecutive or interleaved only by other contract conditions
* Side effects are allowed: we expect only benign side effects to be used.

Points for analysis
-------------------

### Contract-based program alterations

They are not "optimizations" because they can change the semantics of the program.

Sutter says: "otherwise, if such an assumption ever failed, I would be running a different program that is not equivalent to the one I wrote; unlike optimizations which can only reduce the set of possible executions based on facts the compiler can discover about the program, assumptions can expand the set of possible executions by injecting facts not otherwise knowable (practically, or at all) to the compiler"

So, there are two ways to approach the branch in the program that is officially confirmed to have a bug:

* execute what the programmer has wrot in that path.
* apply whatever code alternations that the compiler feels. The goal would be to enable faster program in non-buggy branches.

So there are three categories:
* Correct branch
* Branch taken due to a bug but executed literally
* A no-guarantee on bug-trigerred path.


### Library user decides to run-time check preconditions in the library

The user wants to do it in some parts of his program: in other parts the same precondition on the same function must not be checked. Usage could be:

```c++
void(*fp)(int) = &Library::fun;
fp(1);
```

### Do we need a guarantee that a contract condition will not be evaluated?


### UB in contract conditions

Given these preconditions:

```c++
void fire(Person * p)
  [[pre: p != nullptr]]
  [[pre audit: p->isEmployed()]]; // runtime-checked, continues on failure
```

If there is a way to control the runtime behavior of these checks and the engineer decides that
semantics are "check_and_continue" (a.k.a "inform"), if the first precondition failed, the evaluation of the second would cause UB.

The question is: 

* do we want to magically prevent such UB?
* is this an UB even if contracts are not runtime-checked?


### Zero-overead requirement

When runtime checks are injected into code based on contract annotations, I expect such code to be no slower than if I had manually inserted defensive if-statements.


Definitions
-----------

#### *Contract declaration*

It has a *location* in code: just before the function execution begins, or just after the function execution begins. It has a *predicate*, such as `p != nullptr`. Its "semantics" is, if the execution reaches the contract declaration, and the predicate can be determined (not necessarily by evaluating it) to be `false` then the program has a bug: either before the contract declaration or inside the predicate. The information about a bug in the program can be used in different ways by different tools, including the compiler.  

[Note: the name was chosen so that it does not imply any runtime evaluation of the predicate]


#### Bug

A bug results from an interaction between the code before the contract declaration and the value constraint itself.
The behavior of a program with a bug can differ from one translation to the other based on compiler switches.
This is similar to implementation-defined behavior except that we try to specify in more detail what the variance
in behavior is, and under what conditions.  

=======================

Contracts have more discussions about macros because it is the only feature where we say we will do different things at different times for different people.

UNTRUE: "implementation-defined": character set, include paths

----------------

An assrt becomes a precondition when we refactor some piece of code into a function. we could put assers in an absurdly big numer of places, but we choose to do this sparingly for the places where we expect confusion: middle of a long function or at interface boundarties.

==============================

Questions
---------

1. Can a conforming implementation only type-system-check all the contract conditions and otherwise ignore them?

2. Is it a bug when a function reports "failure" via error code (or any error handling mechanism other than exceptions)
   and the postcondition is violated? (Or, do postconditions need to be week if we signal failures by means ther than
   exceptions?)


