V In cases where people do run with overcommit disabled (or use rlimit), you can actually get NULL returns or std::bad_alloc. 


V Trying to report and handle heap exhaustion (OOM) by throwing a heap-allocated exceptionis inherently problematic. 
  -- nonsense on custom allocator
  
V  propose conditional noexcept
  
 V 4.3.3 proposes exactly what it says it wants to avoid
  
V  std::set_new_handler([]{throw std::bad_alloc();}); -- a proposed solution? What about `noexcept`?

