```c++
constexpr int impl(int num) {
  [[assert: num > 0]] // TODO: use expects/pre
  return num + 42;
}

auto test() {
  // Core constant expression:
  const auto f0 = impl(1);

  // Runtime call to __on_contract_violation:
  const auto f1 = impl(0); 
}
```


The compiler *knows* at compile time that the contract is violated. It would be very useful to get a compile time error at this point. It would also provide a functionality of std::constexpr_assert from P0596.

--------------

Let's assume `constinit` is done via an attribute instead, and you use the attribute and don't get a warning. Does that mean you did it right and the variable is statically initialized, or does it mean the compiler is too old to recognize the attribute and ignored it? Or the compiler does recognize the attribute, but somebody used a flag like -Wno-bad-constinit to turn off the QoI warning? I have to check quite a few things to know whether to trust the compiler (the compiler and version being used, if it supports the attribute, and what compilation flags are active ... which might depend on a user's build system if I'm writing a library for others to use).

If I use a `constinit` keyword it either compiles and does what I want, or it doesn't compile (either because I did something wrong in the code, or the keyword isn't supported).

A soft guarantee is not a guarantee.

-----------------

Exisiting practice: the Standard is not enouh and I have to rely on the compiler to warn me about the potential bugs.

A diagnostic without ill-formedness is a way for adding more safety to C++ while remaining backward compatible at the same time.

[[N2761]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2761.pdf) Jens Maurer, Michael Wong, "Towards support for attributes in C++ (Revision 6)".

attribute + feature macro test == keyword

require diagnostic but not ill-formedness


--------

I propose to differentiate a situation where a program is ill-formed with a diagnostic required from a situation where a diagnostic is required but a program is not ill-formed. The goal (one of) of this would be to allow in the new revisions of the standard a portable detection of a likely buggy constructs, while at the same time preserving backwards compatibility with previous revisions of the standard.
And torpedoing all shops that require code to compile free of warnings.

--------

I do think there are areas where a quality-of-implementation warning is a much better approach than standardizing some particular diagnostic behavior. There are large classes of program problem for which a heuristic can do vastly better than a rigidly-enforced set of standard rules, and compiler vendors are in a much better place to formulate such a heuristic and tweak it to match practical reality. For example, use-of-uninitialized, dangling references, etc.. -- more broadly, anything that's usually thought of as the realm of "static analysis" -- and specific code patterns, such as assignment in a branch condition, or a condition that we can prove always has a particular value. The standard should not invade on those cases; if it does, compiler vendors will presumably try to do what they always try to do: what they think is best for their users, even if that means disregarding the intent of the rule in the standard. Take as an example the C++11 narrowing conversion restriction (that invalidated large amounts of reasonable code due to being a bad heuristic for finding bugs), and note how compilers responded to that: Clang allows the error to be turned off, GCC produces only a warning by default, and ICC doesn't diagnose it at all.

------------

Without a Tony Table, it is hard to determine exactly what this is proposing, since the other situations have no syntactic changes to the status quo.  That being said, it looks like the proposal is asking for mandatory warnings for valid code.

Developers deal with warnings by some combination of:
Rewriting the code to eliminate them
Turning off warnings
Living with noisy builds
In which case:
If you are doing this, nothing changes
This now becomes a "non-standard C++" mode for your compiler
Is a non-starter in many shops (as has already been pointed out), and even if it isn't, shops with noisy builds have a much harder time addressing bugs at compile time because of the noise.
Now, if this proposal is just strongly encouraging but not requiring warnings, I fail to see how that is different than the status quo.

---------

Should compilers give a way for disabling mandatory diagnostics?

`static_warning`

--------------

Narrowing -> if compiler vendors were to provide the definition of "narrowing" they would have not invented the instantiation of templates.
