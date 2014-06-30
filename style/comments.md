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

## Doc comments

Doc comments are prefixed by three slashes (`///`) and indicate
documentation that you would like to be included in Rustdoc's output.
They support
[Markdown syntax](http://daringfireball.net/projects/markdown/)
and are the main way of documenting your public APIs.

### Summary line

The first line in any doc comment should be a single-line short sentence
providing a summary of the code. This line is used as a short summary
description throughout Rustdoc's output, so it's a good idea to keep it
short.

### Sentence structure

All doc comments, including the summary line, should begin with a
capital letter and ending with a period, question mark, or exclamation
point. Prefer full sentences to fragments.

For example:

``` rust
/// Set up a default runtime configuration, given compiler-supplied arguments.
///
/// This function will block until the entire pool of M:N schedulers have
/// exited. This function also requires a local task to be available.
///
/// # Arguments
///
/// * `argc` & `argv` - The argument vector. On Unix this information is used
///   by os::args.
/// * `main` - The initial procedure to run inside of the M:N scheduling pool.
///            Once this procedure exits, the scheduling pool will begin to shut
///            down. The entire pool (and this function) will only return once
///            all child tasks have finished executing.
///
/// # Return value
///
/// The return value is used as the process return code. 0 on success, 101 on
/// error.
```

### Code snippets

> **[OPEN]**

### Avoid inner doc comments.

Use inner doc comments _only_ to document crates and file-level modules:

``` rust
//! The core library.
//!
//! The core library is a something something...
```
