# Rust Atomics and Locks (Mara Bos)

## Basics

```rs
// Leaking a box (p8) - we release ownership of an allocation and promise never drop it
// Comparing to static that means that we can create the allocation in runtime
let x: &'static [i32; 3] = Box::leak(Box::new([1,2,3]));
```

- `Copy` means that when you move them, the original still exists. Similar to integers and boolean.
- Due to interior mutability (for example of Arc that got to change the current count even though it is copied/moved/refereced) it is more appropriate to use words: `shared = &Type` and `exclusive = &mut Type`. Most of the time shared is immutable, but not when we deal with mutithreading. Exclusive ownership doesn't allow to share the same reference. Under the hood it is achieved through `unsafe` code that skips compiler checks.
- All the type with interior mutability are build using `UnsafeCell<T>`.
- `Send` - ownership of the data can be moved to another thread: `Arc<i32>` is send, `Rc<i32>` is not.
- `Sync` - `&data` is Send, that allows to share data with another thread. `i32` is sync, `Cell<i32>` is not.
- `Send` and `Sync` are auto-implemented. To mark that our data doesn't implement any of these traits is to add a marker with type that doesn't have these traits. `PhantomData<Cell<()>>` occupies 0 bytes, doesn't have Sync and it causes a higher level struct also to loose Sync.
- Raw pointers `*const Type` and `*mut Type` are neither Send nor Sync since compiler doesn't know anything about the data they represent. To add back either of them to struct that mentiones pointer use `unsafe impl <Send|Sync> for <StructName>`.
