# Rust Atomics and Locks (Mara Bos)

## Basics

```rs
// Leaking a box (p8) - we release ownership of an allocation and promise never drop it
// Comparing to static that means that we can create the allocation in runtime
let x: &'static [i32; 3] = Box::leak(Box::new([1,2,3]));
```

- `Copy` means that when you move them, the original still exists. Similar to integers and boolean.
- Due to interior mutability (for example of Arc that got to change the current count even though it is copied/moved/refereced) it is more appropriate to use words: `shared = &Type` and `exclusive = &mut Type`. Most of the time shared is immutable, but not when we deal with mutithreading. Exclusive ownership doesn't allow to share the same reference. Under the hood it is achieved through `unsafe` code that skips compiler checks.
