What if a CCS-check has a bug? 
THis could happen if I did not test audit CCSs for a while. Can I disable it w/o assumptions?

Josh's lifecycle: ignore -> check and continue -> check and kill -> assume (all for default)

RYan's example

Josh will never turn on assumes untill he heas not turned runtime checks before.


or invocation of the violation handler and any side effects that function might have.


Natan's use case for axioms as optimizations: test whole program with axiom assumtions -- enough safety.


While it does not seem to be a big issue, I have in my notes this one: " If a function has multiple preconditions, their evaluation (if any) will be performed in the order they appear lexically."
While this is enough to determin


variant in a no-exceptions world: in the end you do not get the variant into the state you wanted , so you have to cancel the operation that needed it anyway.


--------------

[dcl.attr.contract.check]/p6

There are programs that need to resume computation after executing a violations handler. This seems counterintuitive (“continue after a precondition violation!!!?”), but there are two important use cases:

* Gradual introduction of contracts:  Experience shows that once you start introducing contracts into a large old code base, violations are found in “correctly working code.” In other words, contracts are violated in ways that did not cause crashes for the actual use of the code for the actual data used. There are examples where the number of such “currently harmless violations” is massive. The way to cope is to install a violation handler that logs the problem and continues. This allows gradual adoption.

* Test harnesses: Contracts are code, so they can contain bugs. To test contracts, we need to execute examples of violations. A convenient way of organizing such a test suite is to have a violation handler throw an exception and have the main testing loop catch exceptions and proceed running the next test
