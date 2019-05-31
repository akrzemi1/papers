Preconditions, axiom-level contracts and assumptions -- explained
=================================================================

Motivation:

```c++
bool isBetterOffer(shared_ptr<Offer> a, shared_ptr<Offer> b)
{
  return a->price < b.price;
}
```

If etiher is null, the answer is neither `true` or `false`.


Can compiler detect bugs in program logic? (e.g. bad logical operations) It sometimes can when it overlaps with UB.)

Motto:

Any contract declaration divides code into two parts: the "before" and the "after". A contract declaration is an information: if the predicate can be determined to be false at a given point in time, it means that the code before the contract declaration has a bug.

---------------

Mathematical axioms:
 * build therems upon, but cannot make sure that they are true
 * inputs suitable for detectiong contradictions (every contract statement is an axiom in this sense)
 
 
Two levels of usefulness:
* information: what is consided a bug
* how this info can affect code generation

If I were sure that a condition is true, I wouldn't put it...

preconditions are more important than others: a different team will be guaranteeing them and a different one will be declaring them

different meaning of assumptions



contracts are not about "what they do", but "what they tell you".


-----------------

3 modes:

* off
* default
* unspecified


------------------

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
