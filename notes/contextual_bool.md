1. We propose to standardize the existing practice.
2. `requires` does not apply.

http://wiki.edg.com/bin/view/Wg21kona2019/P1401

what converts to `bool`:
* UDTs with explicit (or implicit) conversion to `bool`
* UDTs with an implicit conversion to a scalar type that is convertible to `bool`, such a s a pointer:

  ```c++
  struct X
  {
    operator void*() { return this; }
  };
  ```
* object/hunction pointer
* function name
* reference to a function
* array name
* reference to array
* Pointer to member
* integral type
* pointer to data
* floating-point type
* `nullptr_t`
* unscoped enum
