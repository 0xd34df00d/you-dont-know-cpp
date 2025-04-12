## Is this valid?

```cpp
struct Foo
{
  struct Nested
  {
    bool field = true;
  };

  void doSmth(const Nested& = Nested{});
};
```

Answer: see [this bugzilla entry](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=88165).

## Some covariance

Is this valid?
```cpp
struct Base
{
  virtual const Base* getFoo() { return nullptr; }
};

struct Derived : Base
{
  const Derived* getFoo() override { return nullptr; }
};
```

<details>
<summary>Answer</summary>
Sure: this is covariance in action.
</details>

What about this?
```cpp
struct Base
{
  virtual const Base* getFoo() { return nullptr; }
};

struct Derived : Base
{
  Base* getFoo() override { return nullptr; }
};
```

<details>
<summary>Answer</summary>
Yep, also good: in some sense, `Base*` is a subtype of `const Base*`.
And, of course, `Derived*` would've worked too.
</details>

Now, this is surely valid too, right?
```cpp
struct Base
{
  virtual const int* getFoo() { return 0; }
};

struct Derived : Base
{
  int* getFoo() override { return 0; }
};
```

<details>
<summary>Answer</summary>
Nope: non-class types play by different rules, because otherwise the language would've been too consistent
(see [class.virtual]/8).
</details>

## `constexpr` string literals

Does this compile?

```cpp
constexpr auto f() { return "f"; }
constexpr auto g() { return "g"; }

static_assert(f() == f());
static_assert(f() != g());
```

Here the **answer** is easy:
it's an open question, discussed in [CWG #2765](https://www.open-std.org/jtc1/sc22/wg21/docs/cwg_active.html#2765).

## Maps of non-copyable, non-movable types

Suppose you have a type that's not copyable nor movable, like
```c++
struct ThreadedResource
{
  std::unique_ptr<Resource> handle;
  std::mutex mutex; // mutex isn't move-constructible nor move-assignable nor copyable
};
```
Suppose further you need a hashmap from, say, `int`s to `ThreadedResource`,
with a value potentially missing.

One way is express this is `std::unordered_map<int, std::shared_ptr<ThreadedResource>>`:
a null pointer denotes a missing value.
This is annoying, though, since it has an extra memory allocation and an extra indirection.

Can you do better? In your approach, how would you insert the elements into the map?

<details>
<summary>Potential answer</summary>

One way to do this is to use `std::optional`
(which also expresses the notion of "no value" more explicitly).

Insertions are annoying, though, being a rare example of having both
`std::piecewise_construct` (for the map's `emplace`) and `std::in_place` (for the `std::optional`) in the same expression:

```c++
auto handle = ...;
map.emplace(std::piecewise_construct,
            std::forward_as_tuple(locale),
            std::forward_as_tuple(std::in_place, std::move(handle))
```

Note that the `ThreadedResource` fields are ordered so that `mutex` goes after the `handle`,
so there's no need to pass an initializer to the `mutex`, and it all just works out.
</details>

## When is this function safe or unsafe to use?

```c++
template<auto V>
const auto& foo() { return V; }
```

<details>
<summary>Answer</summary>
It's safe for class types and unsafe for, say, `int`s.
For some reason the standard threats them differently, so

```c++
const auto& v1 = foo<42>();       // bad! dangling reference

struct S { int val; };
const auto& v2 = foo<S { 42 }>(); // fine!
```

Finding the corresponding clauses in the standard is left as an exercise for the reader.
</details>

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

While modern compilers output a warning, what's a reliable and somewhat general way to check functions like this?

<details>
<summary>Hint</summary>

`constexpr` helps.
</details>

<details>
<summary>Answer</summary>

Mark all these functions `constexpr` and try using them in a constant evaluated context, like `static_assert`:
```cpp
static_assert(foo1()[0] == 'G');
static_assert(foo2()[0] == 'G');
static_assert(foo3()[0] == 'G');
```

Say, clang outputs:
```
error: non-constant condition for static assertion
   22 | static_assert(foo3()[0] == 'G');
      |               ~~~~~~~~~~^~~~~~
error: accessing 'str' outside its lifetime
   22 | static_assert(foo3()[0] == 'G');
      |               ~~~~~~~~^
note: declared here
   16 |     const char str[] = "Gotta love C++";
      |                ^~~
```
</details>

---

## Fun with fun templates

What does `bar1` print?
```cpp
template<typename T>
int foo(T) { return 1; }

template<>
int foo(int*) { return 2; }

template<typename T>
int foo(T*) { return 3; }

void bar1()
{
    int test;
    std::cout << foo(&test) << foo<int>(&test) << foo<int*>(&test) << '\n';
}
```

What if we reorder the definitions, as in `bar2`?
```cpp
template<typename T>
int foo(T) { return 1; }

template<typename T>
int foo(T*) { return 3; }

template<>
int foo(int*) { return 2; }

void bar2()
{
    int test;
    std::cout << foo(&test) << foo<int>(&test) << foo<int*>(&test) << '\n';
}
```

Can we still specialize the first template after we've introduced the second one?
<details>
<summary>Answer</summary>

Yep:
```cpp
template<>
int foo<int*>(int*) { return 2; }
```
</details>

---

## What does this print?
```cpp
struct Evil {
  auto begin() { return std::counted_iterator("Library", 7); }
  friend auto begin(Evil&) { return std::counted_iterator("Core", 4); }
  friend auto end(Evil&) { return std::default_sentinel; }
};

Evil rg;
for (char c : rg) { putchar(c); }
std::ranges::for_each(rg, [](char c) { putchar(c); });
```

_borrowed from Arthur O'Dwyer's blog, [where](https://quuxplusone.github.io/blog/2024/12/09/foreach-versus-for/) he also considers this in more detail_
