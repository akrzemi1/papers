13.10.3.1   General [temp.deduct.general]
9.5.3 Deleted definitions [dcl.fct.def.delete]
CWG 2296
https://isocpp.org/files/papers/P2285R0.html


9.5.3/2:

A program that refers to a deleted function implicitly or explicitly, other than to declare it <ins>or to determine if a type or an expression is superficially-invalid during template parameter substitution</ins>, is ill-formed.

13.10.3.1/8:

If a substitution results in a superficially-invalid type or expression, type deduction fails. 

A *superficially-invalid* type or expression *E* formed during the substitution is one that 
would be ill-formed, with a diagnostic required, if written using the substituted arguments.

[*Note*: If no diagnostic is required, the program is still ill-formed. Access checking is done as part of the substitution
process. — *end note*]

Only invalid types and expressions in the immediate context of the function type, its template parameter
types, and its explicit-specifier can result in a deduction failure.

[*Note 6* : The substitution into types and expressions can result in effects such as the instantiation of class template
specializations<del> and/or</del><ins>,</ins> function template specializations <ins>or variable template specializations</ins>, the generation of implicitly-defined functions, etc. Such
effects are not in the “immediate context” and can result in the program being ill-formed. — *end note*]

13.10.3.1/11:

 * <ins>Attempting to reference a deleted function</ins>
 * <ins>Attempting to form an expression used to initialize a default function argument.
   Example:</ins>

```
template <typename T>
void f(T v, T y = T::default_value);

f(1);
```

Table 49:

The expression
`declval<T>() =
declval<U>()` is
<ins>superficially-</ins>well-formed when treated
as an unevaluated
operand (7.2). Access
checking is performed as if
in a context unrelated to T
and U. <del>Only the validity of
the immediate context of
the assignment expression
is considered.
[Note 2 : The compilation of
the expression can result in
side effects such as the
instantiation of class
template specializations and
function template
specializations, the
generation of
implicitly-defined functions,
and so on. Such side effects
are not in the “immediate
context” and can result in the program being ill-formed.
— end note]</del>

https://godbolt.org/z/a4dPP531j
https://github.com/cplusplus/papers/issues/1377
