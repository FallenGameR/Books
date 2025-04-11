# Effective Rust by David Drysdale

## Chapter 1: Types

### Item 3 - Prefer Option and Result transforms over explicit match expressions

![Diagram](attachments/ch1-it4.png)
[Clickable diagram](https://oreil.ly/effective_rust_transforms)

### Item 4 - Prefer idiomatic Error types

- `thiserror` create is useful for automation of writing boilerplate code associated with various errors you got to handle. The generated with it's `derive` errors would not cause consumers of your code to also reference `thiserror` library.
- stack traces are not part of the idiomatic Rust error handling. This aligns with the Rust philosophy that it would rather correctly propagate and handle error than debug it. But if stack traces are needed one can opt-in and use crates like `anyhow` and `backtrace`. But stack traces are costly. `thiserror` and `anyhow` are created by the same author, btw.
- use `thiserror` for libs - errors preserve concrete detailed error information and don't impose extra libs on the users (handled with enums), `anyhow` for apps - to present the error details to the users/devs and to cope with various errors from different libraries (handled with dynamic dispatch)

### Item 6 - Embrace the newtype pattern

- Newtype pattern makes code more robust since it handles unit conversions correctly automatically and there is no ambiguity of redefining behavior for the same traits (abstract classes) by different crates (libs/modules) (the orphan rule). But it causes dev to write boilerplate code for implementing common traits like `#[derive(Debug, Copy, Clone, Eq, PartialEq, Ord, PartialOrd)]` plus `impl fmt::Display`.
- The mitigation of the orphan rule is is a common problem for serialization libs like `serde`. These libs provide some convenience mechanism to handle it.

### Item 7 - Use builders for complex types

- Newtype pattern also causes dev to write boilerplate builder code. There are two conventions:
  - chain-calls to fill in the fields and consume the builder in the last build() call
  - use modifiable builder variable and reply on clone in one or several the build() calls
- It is possible to reduce the boilerplate code by using `derive_builder` crate, but then your lib is adding a dependency on this crate. This seems a type of library that is quite optional. It's like using a Mocks lib in C#.

### Item 8 - Familiarize yourself with reference and pointer types

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
- `Cow` is optimization that holds either borrowed or owned data, it is designed for cases when most of the time you just read the data, but occasionally need to write it (READ-WRITE)

- Smart pointers can look overwhelming `Rc<RefCell<Vec<T>>>`, but they aim to give the right fine-grained pointed semantic
- `Box<T>` - single owner, like `unique_ptr`
- `Rc<T>` - multiple owners, like `shared_ptr` - can leak memory on circular dependencies, but allow you to have self-referencing data structures in the first place
- `Weak<T>` - do not assume ownership to break blocked drops on cyclic data structures, like `weak_ptr`, don't prevent an underlying item to be dropped. One can declare in a tree structure root as weak pointer and children as strong ones. Then as long as the program has a code path that owns the root pointer the whole structure is in memory. But it is still possible to delete children.
- `RefCell<T>` - allow to modify the owned item. With `Rc<T>` that would only be possible if nobody else is able to modify it, like if there is only one user that borrows it.
  - this type allows to violate the borrow checker and move the single ownership check from the compile time to the runtime. Meaning you can modify stuff even with `&self` non-mutable reference. This is called interior mutability in Rust lingo. This is achieved by storing the current number of borrows for the owned item.
  - on API side the user would need to either use `try_borrow` and handle `Result` or assume the borrow would work with `borrow` and be ok that a `panic` would happen if the assumption wrong
- `Cell<T>` is optimization that relies on `Copy<T>` to produce a bit-by-bit valid copy of `T` that is copied back and forth by it's get and set methods. So instead modifying an existing item you always recreate a new one, like `prototype` in JavaScript

- All of the above mentioned smart pointers are for a single thread use only, they can't handle simultaneous access from multiple threads
- `Arc<T>` is multithreaded version of `Rc<T>` that use atomic counters internally, but similarly to `Rc<T>` it doesn't permit internal mutability (compile time check that only one code path can mutate the item)
- `Mutex` ensures that only one thread can mutate the item, similar to `RefCell` it doesn't implement any pointer traits but ha a method that returns `MutexGuard` that does implement them
- `RwLock` is optimization that assumes there could be many readers but only a few writers, makes it similar to `Cow` somehow

### Item 9 - Consider using iterator transforms instead of explicit loops

- In Rust one would see something like `filter(|x| *x % 2 == 0)`. So the `*x` part is necessary visual noise because of the borrowing rules.
- Every collection can be moved into a consuming iterator via `into_iter` that is called automatically in `for( item in collection )` statements. After iterating through the collection that collection is gone. It may be not evident unless the iterated thing can't be copied. If `T` implements `Copy` you may not see that behavior since you would be working on the bitwise items copy.
- Since it is not usually desirable users need to operate not on the items themselves (that would be moved), but on the references to them. And use not `collection.into_iter()` but `(&collection).into_iter()` instead or more succinctly the `collection.iter()` method.
- As a side effect now moved would be not the items but references. And this is ok, but the signatures would need to dereference the items via `*x` syntax.
- `iter` and `iter_mut` methods are the default way to start iterator transformation, C# LINQ analog.

- `flatten` flattens iterator that contains iterators. It it not that common operation for a general type of an iterator though. But actually both `Option` and `Result` impalement iterators that return the item if it exists or nothing when there is nothing or an error. So flatten can be used to filter only successful values.

- aggregating a collection happens through either `reduce(|a, b| a + b)`, `fold(0, |acc, &x| acc + x)` or `scan(0, |state, &x| { *state += x; Some(*state) })`. Reduce uses first element as accumulator, fold uses user specified element, scan can be iterated to get access to the intermediate state of the accumulator.

- iterator transformations likely would produce a faster code compared to traditional for loops since complier can ensure that there is no `[i]` out of bounds exception, but also there are smarts that can optimize away these checks for the for loops as well

- iterator is converted back into a collection via `collect()` call. Collection type must be specified in order for compiler to find needed `FromIterator` trait. E.g. `: Vec<i32>` or `: HashSet<i32>`.
- `collect` can also be used to convert a vector of results into a result that is holding a vector if everything worked fine, that allows to use `?` operator instead of calling panic:

```rs
let result = Vec<u8> = input
  .into_iter()
  .map(|v| <u8>::try_from(v))
  .collect::<Result<Vec<_>,<_>>>()?;
```

## Chapter 2: Traits

### Item 10 - Familiarize yourself with standard traits

- Rust uses traits as generic-based interfaces. There are lots of very basic ones that are similar to copy-constructors, destructors, equality and assignment operators. For these traits the implementation details are boilerplate and Rust provides standard macros. Use the `#[derive(Clone, Copy, Debug, PartialEq, Eq, PartialPrd, Ord, Hash)]` macros when needed.
  - `Clone` runs user-defined code to create value, similar to explicit copy-constructor
  - `Copy` provides bit-by-bit equal value, it is a marker trait for Plain-Old-Data that just inherits `Clone` but actually doesn't call it and the complier is forced to use copy semantic instead of move semantic
  - `Default` creates a usable default value, similar to an explicit default constructor, can be used to fill in long structs with `..Default::default()`, can be used in [builders](#item-7---use-builders-for-complex-types)
  - `Eq` and `PartialEq` allow to compare items, similar to `operator==`, implemented as recursive filed-by-field comparison, `Eq` is a marker trait that assumes reflexivity meaning that that x==x is always true, distinction is important for a `NaN` float type
  - `Ord` and `PartialOrd` allow to compare and order items, implemented as recursive filed-by-field comparison, so the field order matters. `Ord` requires `Eq`, but sometimes `PartialOrd` does make sense - try ordering subsets {1,2} {1,2,4} {1,3} {2,4}
  - `Hash` produces stable hash, needs to provide same have for `Eq` items
  - `Debug` shows item to programmers `{:?}`, similar to `operator<<`, automatically derived, Rust implementation may change between releases, useful to have it by default unless a type contains sensitive information, use `#![warn(missing_debug_implementations)]` lint
  - `Display` shows item to users `{}`, similar to `operator<<`, got be be implemented, developer can enforce stable convention for parsable or localized output

### Item 11 - Implement the Drop trait for RAII patterns

- `drop` doesn't have any return value. So if freeing up some resource may fail a common practice would be to add `release` method that returns `Result`.

### Item 12 - Understand the trade-offs between generics and trait objects

- Generics (trait bounds) are similar to C++ templates with types enforced in runtime. E.g. `where T: Draw` or `<T: Draw>` or `draw: &impl Draw`, although the last one prohibits for the caller to specify the needed override `on_screen::<Circle>(&c)`. Internally compiler produces type-specific code that knows how to work with that specific data that implements the trait bounds. Operations happen on stack and CPU cache is used.
- Trait objects `&dyn Draw` are fat pointers that may reference the data on the heap. They point to the data and to the vtable with the trait methods. Compiler don't write data-specific code. Instead it uses indirection to call the corresponding methods in the runtime. Operations may happen on heap, CPU cache would likely be used only in the case of collections.
- Do not assume that trait objects are significantly slower without a proof of a measured bottleneck
- Generics allow to specify **multiple traits** and thus allow to define conditional behavior for types that implement multiple interfaces. Doing the same thing for trait objects is possible but cumbersome `trait DebugDraw: Debug + Draw {}` and `impl<T: Debug + Draw> DebugDraw for T {}`. Combinations of trait would kill maintainability of this approach.

- This notation is not analog to object oriented inheritance `trait Shape: Draw`. In OOP that means that `Shape-is-a-Draw`. In Rust that notations means `Shape additionally implements Draw`.
- This is implemented by combining several `vtables` from different bounds that a trait implements.
- As a consequence in Rust till 1.86 (April 2025) there was no way to upcast `(BaseDraw) ChildShape` - internal vtables were not composed in a way to support this. And that violated the OOP Liskov substitution principle - be able to replace objects of superclass with objection of subclass.
- But since `1.86` it is possible to do `fn upcast(x: &dyn Sub) -> &dyn Super { x // implicit coercion }`. So a trait object argument `&dyn Shape` can be passed to another function that accepts `&dyn Draw`. Yay!

- Trait objects can't implement `Clone` due to the object safety rules. Trait objects are passed as fat pointer arguments. But clone creates a copy of the object on stack. Rust generally doesn't know at runtime how much space an object would occupy on stack and disallows it. `Box<Self>` theoretically should be safe to return, but as the time of writing the book Rust was 1.7 and [it was not possible](https://github.com/rust-lang/rust/issues/47649).
- In case compiler knows the type size via `Sized` marker trait this restriction is alleviated. But methods that return Self still can't be called.
- Overall Rust tends to prefer generics, and one can say that trait objects deliberately erase their type.
- Dynamically loaded code (via `dlopen`) doesn't have source code and thus generics can't be generated, only trait objects can be used

## Chapter 3: Concepts

### Item 14 - Understand lifetimes

- Lifetimes are fundamentally related to the stack
- Stack holds state relevant to the currently executed function
- Compiler associates every `&` type with a lifetime `'` which sometimes can be hidden (this is called lifetime elision), but it is always `&'lifetime variable` internally
- Lifetime is the period when the value stays at the same address on the stack, meaning `&` references to it would be valid - from value creation to drop or move
- Rust moves values on the stack, from stack to heap and from heap to stack
- For unnamed variables the lifetime behaves as if the temp value is dropped when it is not needed, so `let x = f((a+b)*2);` is expanded to something like:

  ```rs
  let x = {
    let sum = a + b;
    {
      let mul = sum * 2;
      f(mul)
    } // mul drop
  }; // sum drop
  ```

- This is why the following code doesn't compile. Temp variable that holds the function argument is dropped. To fix it this variable needs to be explicitly pinned.

  ```rs
  let var: &Item = return_reference(&mut Item{ value: 42});
  // can't use var.value - referenced Item is dropped just after the function call and before var asignment
  ```




