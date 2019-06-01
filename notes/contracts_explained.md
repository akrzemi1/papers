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


Can compiler detect bugs in program logic? (e.g. bad logical operations) It sometimes can when it overlaps with UB.)

Motto:

Any contract declaration divides code into two parts: the "before" and the "after". A contract declaration is an information: if the predicate can be determined to be false at a given point in time, it means that the code before the contract declaration has a bug.


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
| Mother | Father  | false
| Mother | Child | true
| Father | Child | true  

The second one can be useful in its own right. This also implies that a relation-preserving isomorphism exists between "strict subset" and "is parent" relation. However, it would be a logical error to say that there exists some meaningful relation between set `AB` and person called "Mother", because values from different isomorphisms cannot be mixed arbitrarily.


### Analogies of contracts to mathematical axioms

At least two analogies can be drawm between contract declarations and mathematical axioms.


#### Any contract statement during static analysis

Any contract statement, regardles of if it is a precondition or a postcondition or an assert, regardless if it has level `default` or `audit` or `axiom`, can be used as an input to static analysis. Such analysis can determine if they lead to situations where one of them would be violated (in such case static analyzer would report a warning that program has a bug).
This would be analogous to treating them as a set of axioms in a logical system and determining if this set of axioms is inconsistent.


#### Unchecked contract statements during code generation and execution

The title of this subsection says "checked", but we actually mean semantics `check_and_terminate` from [[P1429r1]][3] (continuation mode off.

-------------------------------------------

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


References
----------

[3]: http://www.open-std.org/JTC1/sc22/wg21/docs/papers/2019/p1429r1.pdf
[[P1429r1]](http://www.open-std.org/JTC1/sc22/wg21/docs/papers/2019/p1429r1.pdf) -- Joshua Berne, John Lakos, "Contracts That Work".
