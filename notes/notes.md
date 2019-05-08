Let's assume `constinit` is done via an attribute instead, and you use the attribute and don't get a warning. Does that mean you did it right and the variable is statically initialized, or does it mean the compiler is too old to recognize the attribute and ignored it? Or the compiler does recognize the attribute, but somebody used a flag like -Wno-bad-constinit to turn off the QoI warning? I have to check quite a few things to know whether to trust the compiler (the compiler and version being used, if it supports the attribute, and what compilation flags are active ... which might depend on a user's build system if I'm writing a library for others to use).

If I use a `constinit` keyword it either compiles and does what I want, or it doesn't compile (either because I did something wrong in the code, or the keyword isn't supported).

A soft guarantee is not a guarantee.

-----------------

Exisiting practice: the Standard is not enouh and I have to rely on the compiler to warn me about the potential bugs.

A diagnostic without ill-formedness is a way for adding more safety to C++ while remaining backward compatible at the same time.
