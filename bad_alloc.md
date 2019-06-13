Document no: D1404r1 <br>
Date: 2019-61-13 <br>
Authors: Andrzej Krzemie&#x0144;ski, Tomasz Kami&#x0144;ski <br>
Reply-to: akrzemi1 (at) gmail (dot) com <br>
Audience: EWG, LEWG


`bad_alloc` is not out-of-memory!
=================================

Paper [[0709R2]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers)
("Zero-overhead deterministic exceptions: Throwing values"), in its optional part 4.3 (about heap exhaustion),
makes an implicit claim that throwing `std::bad_alloc` is synonymous with heap exhaustion and running out of
virtual memory; then it builds the optional parts of the proposal based on this claim.

In this paper we show that there are conditions in the program reported through `std::bad_alloc` that do not represent
out-of-memory. We also show that there are cases of throwing `std::bad_alloc` that are easily reproducible and testable,
and recoverable from.

History
-------

### R0 -> R1

Added more examples. (local allocator doesn't have memory, but global one has plenty. A way to configure in linux no overallocation. -- THanks to Adam)


per-allocation limits
---------------------

Throwing `std::bad_alloc` represents a failure to process a given allocation request for *any* reason. One such reason is
that the requested memory size exceeds a per-allocation limit specified in the system for a given program. Consider the following program that tries to allocate a huge chunk of memory:

```c++
#include <iostream>
#include <vector>
#include <string>

int main()
try {
  std::vector<char> v (1024 * 1024 * 1024); // huge allocation
  std::cout << "OK" << std::endl;
}
catch(std::exception const& e) // bad_alloc handled as any other exception
{
  std::vector<char> s {'E', 'R', 'R', 'O', 'R', ':', ' '}; // reasonable allocation
  std::cout << std::string(s.begin(), s.end()) << e.what() << std::endl;
}
```
Now, in Linux, we set a limit on virtual memory allocation with `ulimit -S -v 204800`, and we run the program. It throws, 
catches and reports `bad_alloc` elegantly. It even allocates memory while handling the exception. This is all fine because the heap has not really been exhausted. This example disproves the claim that allocation failure cases are difficult to test.


No room for a new big block
---------------------------

Sometimes, there is a lot of fragmented RAM available, but no single room to accomodate a particularly big chunk.

Here is the situation with one of the programs in our company. We need to efficiently store huge data structures in RAM, with lots of pointers, and we have, say, 64 GB of RAM on the machines. In order to address this 64GB we require 32 + 4 bits in a pointer, the remaining 28 bits would be a waste. So, in order to avoid waste, we build the apps in 32-bit mode: this way we are able to address 4GB memory with 32-bit pointers and if we run 16 instances on one box, we are able to address the required 64GB of RAM. Of course, the numbers are oversimplified, but everyone should get the picture. Because we are using 32-bit applications we may get allocation failures not because we run out of virtual memory, but because we run out of addressable space. And this is reported cleanly by throwing `std::bad_alloc` on Linux systems: the situation where out-of-memory is signaled upon using memory rather than upon allocation does not apply here.

It happens that while processing some customer requests huge amount of data is generated (which we cannot easily predict ahead of time from just reading the request), and a huge allocation is performed. There is still lots of heap memory available, but because memory is fragmented there is no single chunk to be found capable of storing the required amount of data. This is neatly reported by throwing `std::bad_alloc`. The stack is unwound and and the customer request is reported as failed, but the program still runs and is still able to allocate heap memory (even during the stack unwinding).


Memory allocation failures vs other resource-related failures
--------------------------------------------

The arguments brought in [[0709R2]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers) in favour of not handling memory
allocation failures via exceptions &mdash; difficult to test, often not tested and therefore handled incorrectly &mdash; apply
equally well to any other system resource-related failures. Do you use `std::condition_variable` in your codebase? It can throw `errc::resource_unavailable_try_again` from constructor (if some non-memory resource limitation prevents initialization). Is it easy to reproduce this situation? Are you testing this scenario? If not, your code handles it incorrectly. Does this mean a failure to allocate resources necessary to create a `std::condition_variable` should call `std::terminate()` instead of throwing an exception?

What about running out of file descriptors? Is it easy to test? ...

Of course, memory allocation failure is more special in the sense that one usually needs to use the same resource while
handling exceptions. But this can be managed safely, as we have shown above. Also, once throwing by value is in place there
will be no need to allocate anything while handling exceptions.


Alternatives proposed are inacceptable
--------------------------------------

The alternative proposed in [[0709R2]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers) that our application should switch
to using `try_push_back()` (as proposed in [[P0132R1]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0132r1.html)) or `new (std::nothrow)` instead are not satisfactory: this means we would have to implement manual error handling and propagate it throug all the layers in our application. This is practically impossible:

1. We would now have to manually convey the Boolean information about allocation failure through dozens layers of stack:
   much like error codes are reported in C, which is (as we know) error prone.
2. We would not be able to report allocation failures from constructors. We allocate memory not only with explicit calls
   to `push_back()` but also through copying `std::vector`s and `std::string`s.
   
Ironically, these two above problems are also used as the motivation for
[[0709R]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers), yet what
[[0709R]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers) will force us to do is to replace exception handling with manual
error code propagation in our program.


Separating out-of-memory from other allocation errors
-----------------------------------------------------

It should be possible to tell apart out-of-memory from other allocation failures reported through `std::bad_alloc`. One way is to provide a program-wide constant which represents the memory allocation threshold. A programmer can set it while building the program, or maybe even later, when the program is run. If memory allocation fails for whatever reason and the memory size requested is smaller than the threshold, it means "we cannot allocate even tiny amount of memory, we will likely not be able to even unwind the stack", this can be called heap exhaustion; otherwise (if the caller requested for more memory than the threshold), it may mean an unusually big allocation, which cannot be interpreted as heap exhaustion. Under such distinction, it would be acceptable for us to treat out-of-memory as a fatal error.

But even in this case memory allocating functions would have to be marked as potentially throwing, and the goal indicated by the optional part of [[0709R]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers) of "STL funcitons almost never throw" is still compromised.


Recomendation
-------------

[[P0709]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0709r2.pdf) proposes two significant features:
1. being able to throw cheaply (by value),
2. being required to annotate every potentially throwing function with `try`-operator and therewith being able
   to see in the code any exceptional path.

Our recomendation is to proceed with the first goal and abandon the second. At least drop it from the scope of [[P0709]](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0709r2.pdf). Such explicit `try` operators can be added separately at a later stage. In such alternative future proposal, annotating a potentially throwing function with `try`-operator would be optional, but not putting it on a potentially throwing function could be a compiler warning: compilers do not need the Standard to detect that.


References
----------

* [[P0709R2] Zero-overhead deterministic exceptions: Throwing values ](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0709r2.pdf)
* [[P0132R1] Non-throwing container operations](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2018/p0132r1.html)
