## How are these two functions different?
```c++
template<typename T>
T mkT1() { return {}; }
   
template<typename T>
T mkT2() { return T {}; }
```
   
<details>
<summary>Hint</summary>
Besides the obvious difference in handling of explicit vs nonexplicit default constructors, consider `std::mutex` and C++14 vs C++17.
</details>

---

## Is this code valid?
```c++
struct Foo
{
  int a;

  Foo() = delete;
  Foo(int) = delete;
};
   
int main()
{
  Foo foo1 {};
  Foo foo2 { 10 };
}
```
<details>
<summary>Answer</summary>
Depends on the C++ version.

Up until C++17, both variables are initialized with aggregate initialization. `Foo foo` and `Foo foo(10)` wouldn't be valid, though.

Starting with C++20, this somewhat counter-intuitive behaviour is fixed, and this code no longer compiles.
</details>

---

## (^_=...=_^)

What does this code do, and on what features of C++17 does it rely?
```c++
template<typename F, typename... Ts>
void foo(F f, Ts... ts)
{
  int _;
  (_ = ... = (f(ts), 0));
}
```
<details>
<summary>Answer</summary>
1. It calls the function on the elements of the variadic pack in reverse order.
2. The features are left as an exercise for the reader.
</details>

What problems does this code have, and what should be done to fix them?
<details>
<summary>Hint</summary>
<code>f</code> might return something with an overloaded <code>operator,</code>.
</details>

---

## Conceptual concepts

Assume the following declarations:
```c++
template <typename T>
concept Trivial = std::is_trivial_v<T>;

template <typename T, typename U>
  requires Trivial<T>
void f(T t, U u) { std::cout << 1; }

template <typename T, typename U>
  requires Trivial<T> && Trivial<U>
void f(T t, U u) { std::cout << 2; }
```

Is `f(1, 2)` valid? If yes, what would it print?

1. What if `Trivial<T> && Trivial<U>` is replaced by `Trivial<T> && Trivial<T>` in the second definition?

2. What about `Trivial<T> || Trivial<U>`?

3. What if the definition of `Trivial` gets "inlined", replacing all `Trivial<T>`s with `sd::is_trivial_v<T>`?

---

## I C `memset`

Assume an instance of a `struct` is `memset`ed to zeroes. What would be the value of the padding?\
Further assume a field of that structure is updated. What would be the value of the padding after that field? After other fields?
<details>
<summary>Answer</summary>
Unspecified, unspecified.
</details>

---

## Is this code valid?
```c
char arr[5] = { 0 };
auto pastEnd = arr + 10;
```

What about this one?
```c
char arr[5] = { 0 };
auto pastEnd = arr + 5;
```

---

## Which lines are UB, if any?
```c++
#include <iostream>

struct Foo1
{
    int a;
};

struct Foo2
{
    int a;
    Foo2() = default;
};

struct Foo3
{
    int a;

    Foo3();
};

Foo3::Foo3() = default;

int main()
{
    Foo1 foo11, foo12 {};
    Foo2 foo21, foo22 {};
    Foo3 foo31, foo32 {};

    std::cout << foo11.a << std::endl;
    std::cout << foo12.a << std::endl;
    std::cout << foo21.a << std::endl;
    std::cout << foo22.a << std::endl;
    std::cout << foo31.a << std::endl;
    std::cout << foo32.a << std::endl;
}
```

---

## Is this code valid?
```c++
struct X { int a, b; };
X *make_x() {
    X *p = (X*)malloc(sizeof(struct X));
    p->a = 1;
    p->b = 2;
    return p;
}
```
<details>
<summary>Answer</summary>
Depends on the C++ version, and whether it is C++ to begin with.

Up until C++17, neither an `x` object nor an `int` subobjects are created, and this code is UB.

Starting with C++20, an `x` object and its `int` subobjects are implicitly created, and this code is valid.

It always has been valid C code, though. 
</details>

---

## Is using this function dangerous?
```cpp
auto foo1()
{
    return "Gotta love C++";
}
```

What about this one?
```cpp
auto foo2()
{
    const char *str = "Gotta love C++";
    return str;
}
```

This one?
```cpp
auto foo3()
{
    const char str[] = "Gotta love C++";
    return str;
}
```

<details>
<summary>Answer</summary>
Nope, nope, yep.
</details>

Why? What's the crucial difference between these functions? Is there any difference in their types?

<details>
<summary>Answer</summary>
Accessing any element of the "array" returned by `foo1` and `foo2` is fine.
Try doing that to `foo3` and you'll get an UB, since you'll be using an object whose lifetime has ended!

`foo1` and `foo2` return a pointer to a string that is, roughly speaking, allocated and stored somewhere in the executable at compile time.

The pointer returned by `foo3` references the _local_ array `str` which is initialized by _copying_ that same string.
This array is local to `foo3` and its lifetime ends once the function has returned, hence the UB.
</details>
