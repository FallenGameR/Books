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
- Both can be used in a function that expects a reference to a Point `fn shot(pt: &Point)`. This is achieved by the `Deref` trait implemented for `Box<T>`.
- But the memory for the `Point` in `&Point` is allocated on the stack and it's lifetime is bound to the block that defines it.
- And the memory for the `Point` in `Box<Point>` is allocated on the heap, meaning it would outlive the current block and it would need to be de-allocated elsewhere.

- `AsRef` and `AsMut` allow conversions to a specific type of a reference. For String, for example, `&my_string` would implicitly coerce the type to `&str`. But one can use `AsRef<u8>`, `AsRef<OsStr>`, `AsRef<Path>`, `AsRef<str>`.
- The call would look like `let path = AsRef::<Path>::as_ref(&test);` or shorter `let path: &Path = test.as_ref();`

- `&T` and `Box<T>` don not store information what type they are pointing to, this is ephemeral info known only the the compiler
- Slices `&[T]` and Trait Objects `&dyn T` do store additional type information and thus are called the fat pointers.
- Slices store the length of the collection (+8 bytes on 64-bit machine). Note that slice notation uses include start, exclusive end. So `[2..4]` would have 2 elements `[2]` and `[3]`.
- Slices can't be instantiated directly, but can be casted from either static collection `array` where number of elements known at compile time, or dynamic collection `vector` where lib allocates and deallocates memory based on usage.
- Trait Objects additionally store the pointer to the `vtable` that contains method implementations for the said trait

- `dyn` in signature means dynamic dispatch, meaning that we don't know on runtime what type would it be

- `Borrow<T>` can be used to generalize functions that could work either on references or with moved variables (READ)
- `ToOwned` can be used to generalize functions that could work either on references or with moved variables that become locally owned items (WRITE)
- `Cow` holds either borrowed or owned data, it optimized cases when most of the time you just read the data, but occasionally need to write it (READ-WRITE)

- Smart pointers can look overwhelming `Rc<RefCell<Vec<T>>>`, but they aim to give the right fine-grained pointed semantic
- `Box<T>` - single owner, like `unique_ptr`
- `Rc<T>` - multiple owners, like `shared_ptr` - can leak memory on circular dependencies, but allow you to have self-referencing data structures in the first place
- `Weak<T>` - do not assume ownership to break blocked drops on cyclic data structures, like `weak_ptr`, don't prevent an underlying item to be dropped. One can declare in a tree structure root as weak pointer and children as strong ones. Then as long as the program has a code path that owns the root pointer the whole structure is in memory. But it is still possible to delete children.
- `RefCell<T>` - allow to modify the owned item. With `Rc<T>` that would only be possible if nobody else is able to modify it, like if there is only one user that borrows it.
  - this type allows to violate the borrow checker and move the single ownership check from the compile time to the runtime. Meaning you can modify stuff even with `&self` non-mutable reference. This is called interior mutability in Rust lingo. This is achieved by storing the current number of borrows for the owned item.
  - on API side the user would need to either use `try_borrow` and handle `Result` or assume the borrow would work with `borrow` and be ok that a `panic` would happen if the assumption wrong
- `Cell<T>` relies on `Copy<T>` to produce a bit-by-bit valid copy of `T` that is copied back and forth by it's get and set methods. So instead modifying an existing item you always recreate a new one, like `prototype` in JavaScript

- All of the above mentioned smart pointers are for a single thread use only, they can't handle simultaneous access from multiple threads
- `Arc<T>` is multithreaded version of `Rc<T>` that use atomic counters internally, but similarly to `Rc<T>` it doesn't permit internal mutability (compile time check that only one code path can mutate the item)
- `Mutex` ensures that only one thread can mutate the item



