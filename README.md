1. How are these two functions different?
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

1. Is this code valid?
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
     Yes, in both cases uniform initialization is used. `Foo foo` and `Foo foo(10)` wouldn't be valid, though.
   </details>

1. Assume an instance of a `struct` is `memset`ed to zeroes. What would be the value of the padding?\
   Further assume a field of that structure is updated. What would be the value of the padding after that field? After other fields?

1. Suppose `std::any` is implemented in terms of comparing `std::type_info`. What problems might it have?

1. Is this code valid?
   ```c
   char arr[5] = { 0 };
   auto pastEnd = arr + 10;
   ```

   What about this one?
   ```c
   char arr[5] = { 0 };
   auto pastEnd = arr + 5;
   ```
