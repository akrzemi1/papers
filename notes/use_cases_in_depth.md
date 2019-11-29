Contracts use cases in depth
============================


Beginner's use case
-------------------

We heard, "we want the same experience as with C-assertions". A student writes a test program:

```c++
int fun(int * p)
{
  assert(p);
  return *p;
}
```

And by default -- if the student didn't opt in into defining `NDEBUG` -- when the condition happens
not to hold, she gets a message from the stopped program that it was this very assertion at this
very line with this very expresion that caused the program to stop.

So the core part is: *unless the user opts in into disabling safety checks* runtime checks should be there.

Q: do we want to provide the same user experience from any language UB?

```c++
int fun(int * p)
{
  return *p;
}
```

If `fun()` gets the null pointer, and the user does not opt in into silencing this problem, should she 
get a runtime message? (This is possible with UB sanitizers.)

Q: is it enough to tie it to `-O0`?
