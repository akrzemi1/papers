Preconditions, axiom-level contracts and assumptions -- explained
=================================================================

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

different meaning od assumptions

```c++
bool isBetterOffer(shared_ptr<Offer> a, shared_ptr<Offer> b)
{
  return a->price < b.price;
}
```

If etiher is null, the answer is neither `true` or `false`.

contracts are not about "what they do", but "what they tell you".


-----------------

3 modes:

* off
* default
* unspecified
