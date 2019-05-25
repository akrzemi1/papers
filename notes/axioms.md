Preconditions, axiom-level contracts anf assumptions -- explained
=================================================================

Mathematical axioms:
 * build theories upon, but cannot make sure that they are true
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
