


## [dcl.attr.contract.syn] 
#### Paragraph 6 should be removed as it talks about side UB in declarations of contract statements 
rather than about UB during the evaluations of the conditions:

## [dcl.attr.contract.cond]
#### Add the following at the beginning:

An expression `e` of a type contextually convertible to type `bool` is *deterministic* if every evaluation of expression `bool(e)`
returns the same value. [*Note:* Deterministic expressions can have side effects. *-- end note*]

For a deterministic expression `e`, an expression `pe` is a *predicate equivalent* of `e` if 

* it is deterministic, and
* it has no side effects other than modifications of non-volatile objects whose lifetime 
  began and ended within the evaluation of `pe`, and
* it has the same type as `e`, and
* the value of `bool(pe)` is equal to the value of `bool(e)`.

[*Note:* A deterministic expression can have more than one predicate equivalent. *-- end note*]


#### Modify pargraph 5 so that it reads:


A precondition is checked by evaluating its predicate, or its predicate's predicate equivalent
if the implementation is able to determine one, immediately before starting evaluation of the function body.
[*Note:* (unchanged) *-- end note*]
A postcondition is checked by evaluating its predicate, or its predicate's predicate equivalent
if the implementation is able to determine one, immediately before returning control to the caller of the function.
[*Note:* (unchanged) *-- end note*]

