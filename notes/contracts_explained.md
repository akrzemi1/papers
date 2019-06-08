Preconditions, axiom-level contracts and assumptions -- explained
=================================================================

Motivation
----------

```c++
bool isBetterOffer(shared_ptr<Offer> a, shared_ptr<Offer> b)
{
  return a->price < b.price;
}
```

If etiher is null, the answer is neither `true` or `false`.


Can compiler detect bugs in program logic? (e.g. bad logical operations.) It sometimes can when it overlaps with UB.

Motto:

Any contract declaration divides code into two parts: the "before" and the "after". A contract declaration is an information:
if the predicate can be determined to be `false` at a given point in time, it means that the code before the contract
declaration has a bug. The bug does not have to be *immediately* before contract declaration: it can be far away, but still *before*.


Analogies to mathematical axioms
--------------------------------

C++ intorduces an indentifier with special meaning: `axiom`. Some people think it clearly reflects the semantics due to the analogy to mathematical axioms, while other people are confused and draw conclusions that they have a tool for declaring optimzation hints. In order to clarify this we have to first explain a bit what axioms are in mathematics, and what an analogy is and how it can help and how it can disturb.


### Axioms in mathematics

The most important thing: axioms *do not* state the truth. In fact, no statement in logic or mathematics can be said to be true in the stict sense. Logic offers transformations called inferences that are capable of transforming one postualtes (premises) into other postulates (logical consequences). The only guarantee that these inferences offer is this 'conditional' one: *if* the premises are true *then* the logical consequence it true. But there is no guarantee as to whether the premises alone are true, or if the consequences are true. No mathematical theory is ever "true": it can only say, "if premises we started with happen to be true, then this theory is also true."

This way no true postulate can be ever determined, so in order to break this uncertainty dependency we agree that for some small set of postulates we will not require that they be derived from other postulates. We do not know if they are true or not, but we have to live with it in order to do any progress in logic or mathematics. We call these postulates *axioms*.

Thus, there is a philosophical discomfort connected with axioms, relevant for our analogy: we build theorems, theories, our models of the world based on them, but there is no way for us to determine if these axioms are actually true. There are some properties we can determine for a set of axioms -- that they are inconsistent or that they are insufficient -- but this does not determine their truth or falsehood.


### On analogies

Analogies differ in the level of precision and formalism. On the one side of the spectrum we have statements "oh, this reminded me of something I learned in mathematics"; on the other we have the mathematical notion of *relation-preserving isomorphism*. Let's explain the latter with an example.

We have three values of a floating-point type: `-0`, `+0` and `1`, and a relation "less than", defined by the following table:

| `x`  | `y`  | `x < y` |
|------|------|---------|
| -0   |  0   | false
| -0   |  1   | true
| +0   |  1   | true   

We can transform these values into strings: `-0` to `A`, `+0` to `B` and `1` to `AB`, and relation on values "less than" to relation "strict subset of", defined by the following table:

| `x` | `y`  | `x` is strict subset of `y` |
|-----|------|---------|
| `A` | `B`  | false
| `A` | `AB` | true
| `B` | `AB` | true   

After this transformation the relation preserves the same "characteristics": it returns the same values for the strings as its counterpart for the values corresponding to the strings. This is what we call tthe relation-preserving isomorphism. The benefit we get from defining it is that if for some reason it is easier for us to learn and understand the properties and relations on strings, we can use the isomorphism to later reason about the numeric values. Thus, we are not simply saying "`-0` reminds me of `A`, but we also provide a tool to take our experience and intuition with dealing with strings and apply it numeric values.

A more than one relation-preserving isomorphism can be created for the floating-point values`-0`, `+0` and `1`, for instance the following one:

| `x` | `y`  | `x` is parent of `y` |
|-----|------|---------|
| Mother | Father | false
| Mother | Child  | true
| Father | Child  | true  

The second one can be useful in its own right. This also implies that a relation-preserving isomorphism exists between "strict subset" and "is parent" relation. However, it would be a logical error to say that there exists some meaningful relation between set `AB` and person called "Mother", because values from different isomorphisms cannot be mixed arbitrarily.


### Analogies of contracts to mathematical axioms

At least four analogies can be drawm between contract declarations and mathematical axioms.


#### Any contract statement during static analysis

Any contract statement, regardles of if it is a precondition or a postcondition or an assert, regardless if it has level `default` or `audit` or `axiom`, can be used as an input to static analysis. Such analysis can determine if they lead to situations where one of them would be violated (in such case static analyzer would report a warning that program has a bug).
This makes contract conditions analogous to axioms in a logical system, static analysis analogous to logical inferences, and
detecting errors analogous to determining if a set of axioms is inconsistent.

This analogy is stong as it includes not only one element (axioms), but also others: inferences and axiom inconsistencies.


#### Any precondition during static analysis

One can imagine the following sort of static analysis. When analyzing one function, we try to determine thet when the declared preconditions of the function are met, all control paths cause the functions postconditions to be satisfied. In such analysis, any precondition, regardless if it has level `default` or `audit` or `axiom`, is analogous to a mathematical axiom, any postcondition, regardless if it has level `default` or `audit` or `axiom`, is a theorem derived from the axioms, performing the analysis is analogous to making a set of inferences, and verifying that all postconditions are met is analogous to proving a theorem. 

This analogy is also stong.


#### Unchecked contract statements during code generation and execution

The title of this subsection says "checked", but we actually mean semantics `check_and_terminate` from [[P1429r1]][3]. Suppose we have the following function:

```c++
void fun(X const& x) [[audit: pred(x)]];
```

We compile and run with contract level *default* and continuation mode *off*. Function `fun` declares that it considers it a bug if it is called with the value of x that does not satisfy condition `pred(x)`, and is not required to guarantee anything if such call actually happens. So, the program relies on the fact that `pred(x)` holds, but at the same time under the current build configuration there is no way to runtime-check if this is actually the case. This discomfort resembles the philosophical discomfort of mathematical axioms: we build theorems based on them, but we cannot determine if they are actually true.

We would have the same discomfort if we built the program with contract level *audit* and continuation mode *on*.


#### Contract statements not checked during multiple code generation and execution passes

Going back to the above example:

```c++
void fun(X const& x) [[audit: pred(x)]];
```

The condition is not detectable at run-time when compiled with default mode. But at least there exists a mode in which we can compile it where the condition is evaluated. In this mode we will not be able to test the program in real production environment, but at least the precondition can be runtime checked on *some* data. Thus, our check is relied upon in one build/run, and run-time testsed in another build/run. In contrast to this, contract conditions with level `axiom` are guaranteed never to be runtime-checked in any build/run. The analogy to mathematical axioms is again by resembling the same philosophical discomfort: the condition is relied upon in *all* executions but not runtime-checked in *any* execution.  

The last two analogies are weak, as they only include one elemen (axiom) and nothing else fits into this analogy: what would be an "inference" here? What would a "theorem" be? Or "axiom inconsistency"?

Preprocessing token `axiom` was chosen to reflect the fourth (weak) analogy with mathematical axioms.


#### Incorrect and unintended anaogy to "absolute truths"

We also have to mention one analogy that was never intended, and is absolutely misleading, however, due to the colloquial meaning of "axiom" in everyday speach, is often employed by humans. In the collocquial sense an "axiom" has a property of being "true", or representing "truth". It is more "true" than any other statement that anyone could make. An equivalent of "dogma" with a slightly different emptional baggage.

Under this view, when one sees a declaration containing preprocessing token `axiom`, one is inclined to think that this declaration is equivalent to clang's `__builtin_assume()` or MSVC's `__assume()`.

This interpretation is incorrect, as contract declarations -- regardless of the level -- only declare when a part of program before the declaration contains a bug. They never declare absolute truths about te program state.


Making use of contract declarations
-----------------------------------

The goal of contract declarations is for the programmer to provide an information about the program: when the part of
the program before the contract declaration has a bug. The information is provided in a formal way, so that not only
humans but also automated tools are able to understand it.

Contract declarations can help programmers understand the components they are using and avoid planting bugs in the first place.
They can also assist code reviews: correctness can be assessed more easily, and more bugs can be detected by manual inspection.

Static analyzers, including those embedded in the compilers, can make use of contract declarations to detect potential bugs.

Finally, the presence of the additional information can affect how compilers generate the executable code, in a number of ways. This is based on the important semantic effect of contract declarations: if their condition is evaluated to false (or is determined to be false by other means), the compiler is alowed to modify the observable behavior of the program within certain limits.


### Contract violation handler

One important semantic effect is that under certain build modes the program is allowed to call the contract violation handler when contract condition is determined to be `false`. It is expected of the handler to have side effects, such as logging the violation. Any side effect is by definition a change in semantics, and any such side effect is allowed for the case where the contract condition is `false` (which is equivalent to proving that program has a bug). In fact, under certain build modes the compiler is *required* to call
the violation handler with its side effects.


### Abort

Another characteristic side effect, allowed to be injected when contract condition is determined to be `false` is to abort the program execution. It gives us two things. First, the guarantee that the program determined to have a bug will not continue its execution.
Senond, if the program continues, it means that the condition is true -- verified at run-time -- and program paths after the contract declaration that are reachable only when the condition is `false` can be eliminated. Such elimination can be performed either by the compiler, as an optimization, or by the programmer: he can deliberately choose to neglect the branch outruled by the precondition.


### Optimization hints

The above code path elimination makes the function body potentially faster, when the preconditions are runtime-checked and cause the program to abort. If we reverse this reasoning, this means that disabling the chcks may make the function body slower. We would expect that disabling the run-time checks for contracts should not make the program go slower. Therefore, there is an expectation the same branch elimination inside function body should be allowed even in the situation where contract conditions are not runtime checked.

Such optimization is motivated by the following reasoning. The visible change in behavior will only occur in program paths that contain bugs. Thus correct program parts, remain the same (but faster), whereas parts with bugs will potentially get transformed to something different, with different bugs.

The opposition to such optimizations is based on the observation that there are classes of bugs that the program can deal with,
or whose adverse effect on the program execution are limited and tolerable. The provision to change these "controlled" bugs
into uncontrolled chnges in program behavior can change programs with declared bugs that perform within acceptable limits to those that exceed these limints. The following is an example of a function, taken from [[P1517R0]][4], that copes with the bug:

```c++
void handle_drone(FlightPath *path)
  [[expects LEVEL : path != nullptr]] // for test builds
{
  if (path == nullptr)                // for production builds
    throw flight_error{};
  // ...
}
```

The counter argument to that is that the typical developement process is that one first enables optimizations, which by 
definition changes the semantics of the program, and then performs a series of tests to check if the semantics of the program 
still meet the requirements. This gives sufficient confidence -- obviously, not guarantee -- that the program will operate
within tolerable limits.

Enabling such optimizations is equivalent to runtime-checking the contract predicates and installing the violation handler
with GCC's `__builtin_unreachable()`. 

-----------------


On the other hand, this last expectation is incompatible with the use case in [[P1517R0]][4]:

```c++
void handle_drone(FlightPath *path)
  [[expects LEVEL : path != nullptr]] // for test builds
{
  if (path == nullptr)                // for production builds
    throw flight_error{};
  // ...
}
```

Should contracts in C++ handle this use case?

precoinditions on library interfaces vs preconditions on internal-implementation functions

------------

What id UB is in the contract itself?

--------------------

Contract declarations are associated with certain important semantics in C++: if the 

One thing a compiler can do with the contract declaration is to evaluate its condition when the program is running. 

-------------------------------------------

 
 
Two levels of usefulness:
* information: what is consided a bug
* how this info can affect code generation

If I were sure that a condition is true, I wouldn't put it...

preconditions are more important than others: a different team will be guaranteeing them and a different one will be declaring them

different meaning of assumptions



contracts are not about "what they do", but "what they tell you".

================

contract-based optimizations
------------------------

-----------------

3 modes:

* off
* default
* unspecified


------------------

Timur -> use `audit`

Analogies vs relation-preserving isomorphism: -o, +0, +1 -> A, B, AB 

--------------------




Notes on contracts
------------------

C++ is "unsafe" by design: you can get UB and compiler will not warn you.

Disallowing assumptions is like disallowing vector::operator[] and providing only vector.at() instead.

The goal to "hard to avoid UB" -- something is wrong with this. With CCS-based assumptions we are saying that "violating the contract would be UB" or "we turn bugs into UB, but we add no UB for programs where contracts are respected."

If we agree that static analyzers treat CCSs of all levels as an input, do we still need to insist on axiom-level declaring absolute program-wide truths?

Does someone want axiom-level CCSs as an opt-in for enabling assumptions per CCS?


D1290r2: "Axioms are not really preconditions, postconditions, or assertions but a portable way of spelling assumptions." -> then use a different feature


Instead of axiom axioms we can have `[[unreachable]]` or `[[assume: cond]]`.

make "optimized level" implementation-defined.

need "assumptions" bestandardized? We require of comilers to ignore CCSs, we require that a handler is called, but we do nit require optimizations. We wanly want to allow them: to make them legal. But maybe illegal are better.

Maybe we want to say that contracts are sequenced in order that they appear in.

[[ensures default axiom: x >= 0]] 


References
----------

[3]: http://www.open-std.org/JTC1/sc22/wg21/docs/papers/2019/p1429r1.pdf
[[P1429r1]](http://www.open-std.org/JTC1/sc22/wg21/docs/papers/2019/p1429r1.pdf) -- Joshua Berne, John Lakos, "Contracts That Work".

[4]: http://www.open-std.org/JTC1/sc22/wg21/docs/papers/2019/p1517r0.html
[[P1517R0]](http://www.open-std.org/JTC1/sc22/wg21/docs/papers/2019/p1517r0.html) -- Ryan McDougall, "Contract Requirements for Iterative High-Assurance Systems".
