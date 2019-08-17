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
