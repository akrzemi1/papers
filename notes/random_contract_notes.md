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
