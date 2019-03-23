Appreciate compiler warnings
============================

In this paper we observe that a number of features we are inclined to add to the language whose sole purpose is to cause a potentially buggy program to be ill-formed could be alternatively well handled by compiler warnings. We suggest that instead of adding these features and new keywords, we could shift our focus on clearly communicating where a compiler should issue a warning.


Background
----------


Notes
-----

### narrowing detection
### relational operator chaining
### `constinit`

* http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2019/p1143r1.md
* http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p1143r0.md

### Herb's `throws`

--------------

* use clang only for warnings (doesn't have to link)

"warning as error" to introduce safer subsets of c++.
