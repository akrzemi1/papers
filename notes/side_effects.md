


## [dcl.attr.contract.syn] 
#### First and last sentence in paragraph 6 should be removed as it talks about UB in declarations of contract statements rather than about UB during the evaluations of the conditions. It should read:

An evaluation of a predicate that exits via an exception invokes the function `std::terminate` ([except.terminate]).



## [dcl.attr.contract.cond]
#### Add the following at the beginning:

An expression `e` of a type contextually convertible to type `bool` is *deterministic* if every evaluation of expression `bool(e)`
returns the same value. [*Note:* Deterministic expressions can have side effects. *-- end note*]

For a deterministic expression `e`, an expression `pe` is a *predicate-equivalent* of `e` if 

* it is deterministic, and
* it has no side effects other than modifications of non-volatile objects whose lifetime 
  began and ended within the evaluation of `pe`, and
* it has the same type as `e`, and
* the value of `bool(pe)` is equal to the value of `bool(e)`.

[*Note:* A deterministic expression can have more than one predicate-equivalent. *-- end note*]


#### Modify pargraph 5 so that it reads:


A precondition is checked by evaluating its predicate, or its predicate's predicate-equivalent
if the implementation is able to determine one, immediately before starting evaluation of the function body.
[*Note:* (unchanged) *-- end note*]
A postcondition is checked by evaluating its predicate, or its predicate's predicate-equivalent
if the implementation is able to determine one, immediately before returning control to the caller of the function.
[*Note:* (unchanged) *-- end note*]


## [dcl.attr.contract.check]

#### Modify paragraph 6:

During constant expression evaluation, only predicates of checked contracts are evaluated.
In other contexts:

* predicates of checked contracts are evaluated or their predicate-equivalents are evaluated if implementation is able to 
  determine them.
* for unchecked contracts with *contract-level* different than `axiom`, their predicates are not evaluated;
  it is unspecified whether predicate-equivalents of their predicates, if implementation is able to determine them,
  are evaluated; if such evaluation returns `false`, the behavior is undefined.

#### Add a paragraph at the end:

The predicate `p` of a contract condition with `axiom` *contract-level* is an unevaluated operand. 
If implementation is able to determine the predicate-equivalent of `p`, call it `pe` that does not odr-use any entity referenced in `p` it can evaluate `pe`; if such evaluation returns `false`, the behavior is undefined.
