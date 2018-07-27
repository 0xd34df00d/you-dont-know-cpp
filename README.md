- How is `return {};` different from `return T {};`? What types of `T` would behave differently?
  
  Hint: how does it depend on C++ standard version?
  Hint2: consider `std::mutex`

- Assume an instance of a `struct` is `memset`ed to zeros. What would be the value of the padding?
  Further assume a field of that structure is updated. What would be the value of the padding?

- What if `std::any` is implemented in terms of `std::type_info`. What problems might it have?

- Is this code valid?
  ```c
  char arr[5] = { 0 };
  auto pastEnd = arr + 10;
  ```

  What about this one?
  ```c
  char arr[5] = { 0 };
  auto pastEnd = arr + 5;
  ```
