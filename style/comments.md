# Comments [RFC]

### Avoid block comments.

Use line comments instead:

``` rust
/// Creates and executes a new child task
///
/// Sets up a new task with its own call stack and schedules it to run
/// the provided unique closure.
///
/// This function is equivalent to `TaskBuilder::new().spawn(f)`.
pub fn spawn(f: proc():Send) { ... }
```

### Style doc comments like sentences.

Doc comments should begin with capital letters and end in a period,
even in the short summary description. Prefer full sentences to
fragments.

> **[FIXME]** Example.

### Avoid inner doc comments.

Use inner doc comments _only_ to document crates and file-level modules:

``` rust
//! The core library.
//!
//! The core library is a something something...
```
