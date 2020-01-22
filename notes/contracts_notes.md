Random notes on contracts
=========================

do we want same student's experience for [[assert: c]] and language UB for pointer dereference (through UB-sanitizer?

asserts that act like preconditions

by introducing preconditions to my code, I am potentially introducing bugs, as the prdicates can have UB. The goal of disabling assertions in production may be to avoid bugs

substitutability: one function can meet two different contracts (measured as pair of pre and postcondition)


If I wat to wrap my `std::array<int, a>` into a non-template class/function, I may want `[[pre assume]]` to guarantee the same compiler optimizations as before the wrap.

Document tree precondition use-cases.

Document use case for `__builtin_assume()`.

Document time travel optimizations.
