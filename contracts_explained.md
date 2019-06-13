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
declaration has a bug. The bug does not have to be *immediately* before contract declaration: it can be far away, but still *before*. Passing an invalid argument to a function is not necessarily a bug itself: it is a symptom of a bug that may be somewhere else.

------------------



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

One important semantic effect is that under certain build modes the program is allowed to call the contract violation handler when contract condition is determined to be `false`. It is expected of the handler to have side effects, such as logging the violation. Any side effect is by definition a change in semantics, and any such side effect is allowed for the case where the contract condition is `false` (which is equivalent to proving that program has a bug). In fact, under certain build modes the compiler is *required* to call the violation handler with its side effects.


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
  [[expects LEVEL : path != nullptr]] // for static analysis and test builds
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

All the above things, however, that a compiler can do with the information in contract declarations is secondary. The primary 
goal of contract declarations is to provide the information: that a program that caused the contract condition to evaluate to `false` has a bug somewhere before the contract declaration. In other words, the main usefulness of contract declarations is not how they affect the code generation, but what they tell us about the program.


Different meanings of word "assume"
-----------------------------------

In discussions on contracts words "assume" and "assumption" have a number of different meanings. In this section we try to list the different meanings.


### Input to static analysis

This is in the context of performing static analysis. Suppose some function f(), whose both declaration and definition is seen, but not its usages, is analyzed in order to detect potential bugs:

```c++
Numeric f(Numeric x, Numeric y)
  [[expects : x >= Numeric::zero()]]
  [[ensures r: r >= Numeric::zero()]]
{
  if (meets_criteria(y))
    return Numeric::one();
  else
    return x;
}
```
 
Because the function has a postcondition, one thing to check is if this postcondition will be satisfied. We do not know what function `meets_criteria()` does, we only know its declaration:

```c++
bool meets_criteria(Numeric n); // by-value, wide contract
```

But this information is not necessary to achieve our task. We know that function f() either returns 1, or it returns `x`. And of `x` we know that the precondition states that it is non-negative. We do not know if in the real program this `x` will be non-negative. We could get a negative value, but this does not affect the static analysis. Static analysis is only is interested in what is stated in the precondition. The outcome of static analysis is, "if preconditions of this function are met on entry, I confirm that this function will meet its postcondition." That is, there is no claim being made if the precondition is actually satisfied or not.

In the light of this, we could claim that contract statements are analogous to mathematical axioms. By saying this, we refer to the following property of the mathematical axioms:

In a given system we never claim that a given axiom is actually true. We only say that *if* it (and the remaining axioms) is true and consistent with other axioms, whatever statement we draw from this axiom will also be true. So the mathematical axioms do not state "absolute truths": they only state relations between statements.

Thus, making the statement like, "if condition C1 is true, then condition C2 is true also" can be seen as "assuming" that C1 is true. But it is not the same as saying "C1 is true".

Note that contract statements in the example above do not have axiom level. Because this is contracts statements of *any* level that are analogous to mathematical axioms: not only the axiom-level ones.

For the lack of a better name, I will call this shade of assumption a correctness-proof-assumption.


### runtime-verified conditions

I have seen this one in the responses from Daniel. This is the case when a given contract statement is compiled in a mode where the condition is evaluated at run-time and upon false result causes the program to terminate (or an exception to be thrown). In John's and Josh's terminology, this is the check_never_continue semantic. Inside the body of such function, if the condition C from the contract statement appears also in the body of the function, this second appearance is *redundant* and it is perfectly legal, and in fact desired, for the compiler to remove it:

```c++
// compiled with default build level (or check_never_continue semantic)
Numeric f(Numeric x)
  [[expects: x >= Numeric::zero()]]
{
  if (x < Numeric::zero())    // this will be elided
    return Numeric::zero(); // this will be elided

  return some_algo(x);
}
```

We can say that with check_never_continue semantic of the contract statement, the compiler can *assume* that the condition in the contract statement holds inside the function body. This is analogous to the situation where the condition in the if-statement (which is run-time checked) can be elided if it reappears inside the block controlled by the if-statement:

```c++
if (x >= 0)
{
  if (x < 0)                 // will be elided
    throw invalid_argument{""};  // will be elided
  return sqrt(x);
}
```

I will refer to this as if-statement-assumption.


### Code generation hint

This is in the context of the UB-based optimizations. When a given function f(), whose declaration we can see but not the definition, is invoked in another function u(), and function f() has a precondition statement, and the build mode is off (condition can still be evaluated and if it returns false we have UB) (this is equivalent to John's and Josh's `assume` semantic), the compiler, inside function u(), can procede as if there were a phantom if-statement in the top of the branch that directly leads to calling function f():

```c++
// compiled with build level off, or `assume` semantic
void f(Numeric x) [[expects: Numeric x >= Numeric::zero()]];

void u(Numeric x, Numeric y)
{
  if (x < Numeric::zero()) // never elided
    log(x);  // never elided

  if (y >= Numeric::zero())
  {
    // if we get here, we will call f() inevitably
    //  compiler can behave as if the following code has been executed at this point
    // if (x < Numeric::zero()) abort();
    // because of the subsequent call to f()

    if (x < Numeric::zero()) // can be elided (because phantom if-guard was invoked)
      log(x);  // can be elided

    f(x);

    f (x < Numeric::zero()) // can be elided
      log(x);  // can be elided
  }
}
```

In the above example `log()` represents a function that returns in a normal way: does not throw, does not terminate the program, does not do a "long jump".

Again, for the lack of a better name I will refer to it as optimization-assumption.


### Code path omitted by the programer

This can be illustrated with the following simple example.

```c++
int f(int * p)
{
  return *p;
}
```

The author of this function chose not to specify what happens if `p` is null. Either he controls all the places where the
function is invoked (possible for private member functions or functions in anonymous namespaces) and sees that null value
is never passed, or he is confident that all potential callers understand his constraints and that they will obay to these constraints.


Analogies to mathematical axioms
--------------------------------

C++ intorduces an indentifier with special meaning: `axiom`. Some people think it clearly reflects the semantics due to the analogy to mathematical axioms, while other people are confused and draw conclusions that they have a tool for declaring optimzation hints. In order to clarify this we have to first explain a bit what axioms are in mathematics, and what an analogy is and how it can help and how it can disturb.


### Axioms in mathematics

The most important thing: axioms *do not* state the truth. In fact, no statement in logic or mathematics can be said to be true in the stict sense. Logic offers transformations called inferences that are capable of transforming one postualtes (premises) into other postulates (logical consequences). The only guarantee that these inferences offer is this 'conditional' one: *if* the premises are true *then* the logical consequence it true. But there is no guarantee as to whether the premises alone are true, or if the consequences are true. No mathematical theory is ever "true": it can only say, "if premises we started with happen to be true, then this theory is also true."

This way no true postulate can be ever determined, so in order to break this uncertainty dependency we agree that for some small set of postulates we will not require that they be derived from other postulates. We do not know if they are true or not, but we have to live with it in order to do any progress in logic or mathematics. We call these postulates *axioms*.

Thus, there is a philosophical discomfort connected with axioms, relevant for our analogy: we build theorems, theories, our models of the world based on them, but there is no way for us to determine if these axioms are actually true. There are some properties we can determine for a set of axioms -- that they are inconsistent or that they are insufficient -- but this does not determine their truth or falsehood.


### On analogies

Analogies differ in the level of precision and formalism. On the one side of the spectrum we have statements "oh, this reminded me of something I learned in mathematics"; on the other we have the mathematical notion of *relation-preserving isomorphism*. Let's explain the latter with an example.

We have the red blood cell compatibility table:  

| donor --> <br> recepient | O- | O+ | A- | A+ | B- | B+ | AB- | AB+
---------------------------|----|----|----|----|----|----|-----|----
| 0-                       | ok |    |    |    |    |    |     |    
| 0+                       | ok | ok |    |    |    |    |     |    
| A-                       | ok |    | ok |    |    |    |     |    
| A+                       | ok | ok | ok | ok |    |    |     |    
| B-                       | ok |    |    |    | ok |    |     |    
| B+                       | ok | ok |    |    | ok | ok |     |    
| AB-                      | ok |    | ok |    | ok |    | ok  |    
| AB+                      | ok | ok | ok | ok | ok | ok | ok  | ok 


We could represent the same as a relation on numbers in binary representation: if for every bit position in *lhs* the bit value is less or equal than the corresponding bit value in *rhs* then the relation returns true:

| *rhs* -> <br> *lhs* | 000 | 001 | 010 | 011 | 100 | 101 | 110 | 111
----------------------|-----|-----|-----|-----|-----|-----|-----|----
| 000                 | yes |     |     |     |     |     |     |    
| 001                 | yes | yes |     |     |     |     |     |    
| 010                 | yes |     | yes |     |     |     |     |    
| 011                 | yes | yes | yes | yes |     |     |     |    
| 100                 | yes |     |     |     | yes |     |     |    
| 101                 | yes | yes |     |     | yes | yes |     |    
| 110                 | yes |     | yes |     | yes |     | yes |    
| 111                 | yes | yes | yes | yes | yes | yes | yes | yes

That is, we map "O-" onto `000`, "O+" onto `001`, and so on, and finally we map relation "recepient with blood type *x* can receive blood from donor with blod type *y*" onto relation "for each bit posiiton *lhs* has smaller or equal value than *rhs*". 

After this transformation the relation preserves the same "characteristics": it returns the same values for the blood types as its counterpart for the integral values corresponding to the blood types. This is what we call tthe relation-preserving isomorphism. The benefit we get from defining it is that if for some reason it is easier for us to learn and understand the properties and relations on integers, we can use the isomorphism to later reason about blood types. Thus, we are not simply saying "`0+` looks similar to number `0`", but we also provide a tool to take our experience and intuition with dealing with numbers and apply it to blood cell types.


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

We also have to mention one analogy that was never intended, and is absolutely misleading, however, due to the colloquial meaning of "axiom" in everyday speach, is often employed by humans. In the collocquial sense an "axiom" has a property of being "true", or representing "truth". It is more "true" than any other statement that anyone could make. An equivalent of "dogma" with a slightly different emotional baggage.

Under this view, when one sees a declaration containing preprocessing token `axiom`, one is inclined to think that this declaration is equivalent to clang's `__builtin_assume()` or MSVC's `__assume()`.

This interpretation is incorrect, as contract declarations -- regardless of the level -- only declare when a part of program before the declaration contains a bug. They never declare absolute truths about te program state.


Concerns about UB
-----------------


### UB-based optimizations

In today's C++ UB can be used for optimizations. The following example illustrates how an UB inside one function (`f`)
can alter the body of another function (`g`):

```c++
int global_error_count = 0;

void log_error() { 
  ++global_error_count;  // this has side effect
}                        // but is guaranteed to return normally

int f(int* p)
{
  // implicit assumption: p != nullptr
  return *p;
}
  
int g(int* p)
{
  if (p == nullptr)     // this will get ellided
    log_error();        // due to the implicit assumption

  return f(p);
}
```

The author of funcition `f` has made an assumption: it will never receive a null pointer. As a consequence, the function is
written in such a way that passing it a null pointer will be UB. Now, function `g` checks if the pointer is null and if so, it "logs" this fact. However it logs it in a specific way: the function never throws and never stops the program via `std::exit()` or `std::abort()` or similar: it is guaranteed to return normally. (`noexcept` is not needed if the compiler can see the function body.) But then it unconditionally calls `f(p)`. If `p` is null, then the call to `f(p)` will be UB; since compiler is allowed to
arbitrarily change the meaning of the program on UB, even prior to the UB event, it can eliminate the `if`-statement altogether:
we would only see its effects if the program hit UB the moment later. This removal obviously changes the visible effects of the program, but only in the path that has UB. In the other path it makes the program run faster because no condiiton in `if`-statement needs to be evaluated. THis is called a "time travel" optimization, and is often found surprising as the effects of UB  precede the UB. Clang 6 does perform this optimization in `-O3`.

An important thing to stress here is that this is the present behavior in C++. Function `f()` above has a precondition,
even though there is no means to express it. And this precondition causes the body of function `g()` to be altered in a potentially surprising way. If contract statements are added to C++, even with UB-based optimizations, they do not introduce
any new kind of dngerous optimizations: they only make these optimizations explicit and more easily detectable.


### Contract-induced UB

Under the current specification, if a contract statement is not checked but is nonetheless determined to be `false` the implementation has the freedom to choose any behavior it feels appropriate. This gives the room for implementations to offer
useful contract-related features not specified in the standard: 

1. Refuse to compile programs that inevitably lead to contract violation.
2. Enable bug reporting through UB-sanitizer.
3. Selectively runtime-check contract statements, e.g. only default-level preconditions, as described in [[P1421R0]][5].
4. Offering an alternative, more configurable, mechanism for installing custom callbacks for cases where a contract condition is violated.
5. Arbitrarily cnange the behavior of the program in the "negative" path in order to make the "positive" paths run faster. This is an UB-based optimization.
6. Do nothing, as though there was no contract statement.


### Unintended program modifications

The fact that the behavior for non-runtime checked contracts is not defined makes the following use case for contract
statements impossible, at least in a portable (implementation-independent) way:

```c++
void handle_drone(FlightPath *path)
  [[expects LEVEL : path != nullptr]] // for static analysis and test builds
{
  if (path == nullptr)                // for production builds
    throw flight_error{};
  // ...
}
```

This concern could be addressed by requiring another two-state switch of implementations that controls what happens when unchecked contract evaluates to `false`:  whether nothing happens or undefined behavior.

However, it should be noted that even if this is fixed, contract statements can cause arbitrary code modifications in unexpected ways; e.g., when there is an UB in the contract condition itself:

```c++
void f(X* x) [[expects: x->p()]];

void g(X* x)
{
  if (x == nullptr)
    record_bug();   // can be elided
  
  f(x);
}
```

In the above example, `f()` has an explicit precondition, but it also has an implicit one that `x` is not null: either the programmer forgot to type it, or he considered it so obvious that he didn't even consider it worthwile to write it down. If this is compiled in the default build mode, the `if`-statement in function `g()` may be elided. 

It should be noted that contract statements do not magically address all problems with program safety.

----------------------------

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

What if UB is in the contract itself?

--------------------
AB- https://wandbox.org/permlink/NnGCJd586fILCQXo

https://wandbox.org/permlink/Uvg38P69zZPbagGa

-------------------------------------------




If I were sure that a condition is true, I wouldn't put it...

preconditions are more important than others: a different team will be guaranteeing them and a different one will be declaring them

different meaning of assumptions



contracts are not about "what they do", but "what they tell you".

================

contract-based optimizations
------------------------



------------------

Timur -> use `audit`: https://godbolt.org/z/c5qvBj

```c++
void f (float* data, std::size_t size)
  [[expect audit: size % 8 == 0]]
{
    for (int i = 0; i < size; ++i) 
        data[i] = std::clamp(data[i], -1.0f, 1.0f);
}
```

Analogies vs relation-preserving isomorphism: -o, +0, +1 -> A, B, AB 

--------------------

### surprising effects:

```c++
void f(X* x) [[expects: x->p()]];

void g(X* x)
{
  if (x == nullptr)
    record_bug();   // can be elided
  
  f(x);
}
```


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

[5]: http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1421r0.md
[[P1421R0]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1421r0.md) -- Andrzej Krzemieński, "Assigning semantics to different Contract Checking Statements".
