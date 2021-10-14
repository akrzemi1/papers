
Expressions in contract annotations are looked up as if they appear in `noexcept` specifications of the functions they appertain to.
 
  1. initialize function parameters (including default function arguments)
  2. **perform precondition test sequence** (inspecting function argument objects)
  3. start function call (noexcept barrier)
  4. **perform precondition test sequence** (inspecting function argument objects)
  5. run function body
  6. initialize the return object
  7. call destructors of automatic objects
  8. **perform postcondition test sequence** (inspecting function argument objects, and return object)
  9. end function call (noexcept barrier)
 10. **perform postcondition test sequence** (inspecting function argument objects, and return object) 
 11. call destructors of function parameters
