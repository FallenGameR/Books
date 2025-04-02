# Effective Rust by David Drysdale

## Chapter 1: Types

### Item 3 - Prefer Option and Result transforms over explicit match expressions

![Diagram](attachments/ch1-it4.png)
[Clickable diagram](https://oreil.ly/effective_rust_transforms)

### Item 4 - Prefer idiomatic Error types

- `thiserror` create is useful for automation of writing boilerplate code associated with various errors you got to handle. The generated with it's `derive` errors would not cause consumers of your code to also reference `thiserror` library.
- stack traces are not part of the idiomatic Rust error handling. This aligns with the Rust philosophy that it would rather correctly propagate and handle error than debug it. But if stack traces are needed one can opt-in and use crates like `anyhow` and `backtrace`. But stack traces are costly. `thiserror` and `anyhow` are created by the same author, btw.
- use `thiserror` for libs - errors preserve concrete detailed error information and don't impose extra libs on the users (handled with enums), `anyhow` for apps - to present the error details to the users/devs and to cope with various errors from different libraries (handled with dynamic dispatch)

## Item 6 - Embrace the newtype pattern

- Newtype pattern makes code more robust since it handles unit conversions correctly automatically and there is no ambiguity of redefining behavior for the same traits (abstract classes) by different crates (libs/modules) (the orphan rule). But it causes dev to write boilerplate code for implementing common traits like `#[derive(Debug, Copy, Clone, Eq, PartialEq, Ord, PartialOrd)]` plus `impl fmt::Display`.
- The mitigation of the orphan rule is is a common problem for serialization libs like `serde`. These libs provide some convenience mechanism to handle it.

## Item 7 - Use builders for complex types

- Newtype pattern also causes dev to write boilerplate builder code. There are two conventions:
  - chain-calls to fill in the fields and consume the builder in the last build() call
  - use modifiable builder variable and reply on clone in one or several the build() calls
- It is possible to reduce the boilerplate code by using `derive_builder` crate, but then your lib is adding a dependency on this crate. This seems a type of library that is quite optional. It's like using a Mocks lib in C#.

## Item 8 - Familiarize yourself with reference and pointer types

- Both `&Point` and `Box<Point>` are pointers that occupy 8 bytes of space on a 64-bit machine.
- Both can be used in a function that expects a reference to a Point `fn shot(pt: &Point)`.
- But the memory for the `Point` in `&Point` is allocated on the stack and it's lifetime is bound to the block that defines it.
- And the memory for the `Point` in `Box<Point>` is allocated on the heap, meaning it would outlive the current block and it would need to be de-allocated elsewhere.
