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

