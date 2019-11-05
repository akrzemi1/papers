Anthony's example: Identifiers used to be strings. We changed them to `int`s and hoped that code that used them would be
now compiler-error, but the assignment from `int` to `string` works with surprising results.

Assignment from float to string is never good idea

```c++
if (cond1) {
  str = 'A';
}
else if (cond2) {
  str = 'B';
}
else {
  str = 0;  // I mean '\0'
}
```


RU013: https://github.com/cplusplus/nbballot/issues/13

The assignment from `char` is already fishy itself.

Potential use cases are when we know we mean a character, but the typesystem forces it to be `int`.

clang-tidy finds this bugs. Maybe rely on errors.

A mechanical fix exists add `int_to<CharT>`: `str = int_to<char>('A' + i);`
