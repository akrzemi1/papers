Related guidelines
------------------

### Avoid "overlapping" constructors

Do not declare constructors that can be called with the same set of parameter types. 

#### Rationale

If you do so, you may not be easily be able to tell which of these constructors is selected, and the selection may chage as you
change the syntax used for initialisation

#### Example

```c++
class A  // non-compliant: "overlapping" constructors
{
public:
  explicit A(int x); // #1
  A(long x);         // #2
};

void example_1()
{
  A a1 (1); // chooses constructor #1
  A a2 = 1; // chooses constructor #2
}

template <typename T>
class Vec  // non-compliant: "overlapping" constructors
{
public:
  Vec(int size, T value);                 // #1
  Vec(std::initializer_list<int> values); // #2
};

void example_2()
{
  Vec<int> v1(3, 2); // chooses constructor #1
  Vec<int> v2{3, 2}; // chooses constructor #2
}

class B  // non-compliant: "overlapping" constructors
{
public:
  B(const B& b) // #1 (copy constructor)
    
  template <typename T>
    B(T&& x);   // #2 (perfect-forwarding constructor)
};

void example_3(const B cb, B mb)
{
    B b1(cb); // chooses constructor #1
    B b2(mb); // chooses constructor #2
}

```

#### Note

`std::vector` violates this guideline.


### Avoid using braces for type with `initializer_list` constructor

Avoid using braces for initialization if there is a chance that the type you are initializing might be providing an `initializer_list` constructor. If in doubt assume that it does.


#### Rationale

In case types like STL containers (or types that try to provide a similar interface) provide overlapping constructors the one that gets chosen may not be the one you think.

#### Example

```c++
constexpr size_t Width = 80;
constexpr char Symbol = '-';
std::string horizontal_line {Width, Symbol}; // non-compliant
// horizontal_line.size() == 2

std::string horizontal_line_2 (Width, Symbol); // compliant
// horizontal_line_2.size() == 80

template <typename T>
std::vector<T> make_vec()
{
  constexpr size_t InitialSize = 100;
  std::vector<T> vec {InitialSize}; // non-compliant
  return vec;
}

auto v1 = make_vec<std::string>(); // v1.size() == 100
auto v2 = make_vec<int*>();        // v2.size() == 100
auto v3 = make_vec<char>();        // v3.size() == 1
```

#### Note

Even though brace initialization has the potential to detect narrowing at compile time, it is still better not to use brace initialization in order to avoid other bugs. Narrowing can be detected by other static analysis tools such as clang-tidy.

#### Note

Aggregate types (without user-defined constructors) and built-in scalar types by nture do not have `initializer_list`-constructors, so they can be used with brace initialization.
