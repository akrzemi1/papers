`try`-expressions: no hidden paths
----------------------------------

Safety-criticl systems requirement: you have to assume there is a dozen bugs in your function. 
Is it harer to detect them if there are invisible control flow paths.

Can we fix this with attributes?


Efficiency/implementability of today's exceptions
-------------------------------------------------

### Requiring TLS

Maybe make some exception features conditionally supported:

* `throw;` outside of `catch`
* `std::uncaught_exceptions()`
* `std::current_exception()`

Introduce `catch(..., std::exception_ptr p)` syntax.


### Upper bound on worst case timing:

Maybe catch by value. Use `final` types.


### Herbxceptions not implementable

```c++
class Tool {/***/};
static_assert(sizeof(Tool == 4));

Tool x;
Tool y;
Tool z;
new (&y) Tool{makeTool()};
```
