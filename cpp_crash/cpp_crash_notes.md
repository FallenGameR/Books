# C++ Crash Course by Josh Lospinoso

## Chapter 1

- `std::size` is used instead of sizeof(array)/sizeof(int)
- Mutiple lines can be concatenated in a readable way:

  ```cpp
  auto info =
    "first line"
    "second line"
    "I wonder how newlines are handled";
  ```

- `{}` gives you the most unabgious initialization. There case cases when `()` in calling constructors behaves strangelly. So this is the preffered way of initializing everything:

  ```cpp
  auto number{42};
  auto CurrentTime{Time::Now};
  auto numbers[4]{1,2,3,4};
  ```

- It's better to use `enum class`, not `enum` since the first one enforces always using the enum name. That leads to less errors in the code.
