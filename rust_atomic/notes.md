# Rust Atomics and Locks (Mara Bos)

## Basics

```rs
// Leaking a box (p8) - we release ownership of an allocation and promise never drop it
// Comparing to static that means that we can create the allocation in runtime
let x: &'static [i32; 3] = Box::leak(Box::new([1,2,3]));
```
