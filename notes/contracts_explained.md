--------------------
AB-  https://wandbox.org/permlink/NnGCJd586fILCQXo

https://wandbox.org/permlink/Uvg38P69zZPbagGa

--------------------------------

What if UB is in the contract itself?

----------------

contracts are not about "what they do", but "what they tell you".


If I were sure that a condition is true, I wouldn't put it...




------------




preconditions are more important than others: a different team will be guaranteeing them and a different one will be declaring them

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



The need for providing optimization hints
-----------------------------------------

The existence of compiler specific extensions for specifying optimization hints, such as `__builtin_assume()` in Clang or `__assume()` in MSVC, shows that there is a need for this feature. Because C++ does not offer (yet) a way to portably express this, programmers get inclinde to (ab)use contract statements for this purpose. For instance, consider this example from Timur Doumler: 

```c++
void f (float* data, std::size_t size)
  [[expect audit: size % 8 == 0]]
{
    for (int i = 0; i < size; ++i) 
        data[i] = std::clamp(data[i], -1.0f, 1.0f);
}
```

If the compiler can know that the size of the array is a multiple of 8, it can generate a more efficient code. Given the current contracts specifications in default level this contract statement can be used for optimization purposes. Additionally, in audit mode, where we accept the slowdown, the condition can be run-time verified. 

This technically fits into the definition of a contract checking statement: if the condition is not true upon function entry, then there is a bug before the function call. However, 

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


Instead of axiom axioms we can have `[[unreachable]]` or `[[assume: cond]]`.

make "optimized level" implementation-defined.

need "assumptions" bestandardized? We require of comilers to ignore CCSs, we require that a handler is called, but we do nit require optimizations. We wanly want to allow them: to make them legal. But maybe illegal are better.

Maybe we want to say that contracts are sequenced in order that they appear in.

[[ensures default axiom: x >= 0]] 




Notes on contracts
------------------

C++ is "unsafe" by design: you can get UB and compiler will not warn you.

Disallowing assumptions is like disallowing vector::operator[] and providing only vector.at() instead.

The goal to "hard to avoid UB" -- something is wrong with this. With CCS-based assumptions we are saying that "violating the contract would be UB" or "we turn bugs into UB, but we add no UB for programs where contracts are respected."

If we agree that static analyzers treat CCSs of all levels as an input, do we still need to insist on axiom-level declaring absolute program-wide truths?

Does someone want axiom-level CCSs as an opt-in for enabling assumptions per CCS?


D1290r2: "Axioms are not really preconditions, postconditions, or assertions but a portable way of spelling assumptions." -> then use a different feature
