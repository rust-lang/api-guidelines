# Modules

> **[OPEN]** What general guidelines should we provide for module design?

> We should discuss visibility, nesting, `mod.rs`, and any interesting patterns
> around modules.

## Basic design

> **[OPEN]** This documents the simple, common pattern of module
> design - but others exist and improvements are appreciated.

The file `mod.rs` in a module defines the base-level imports of the
module. For all except trivial modules (and
[test cases](../testing/README.md)), it is better to keep this in a
separate file.

A big use of `mod.rs` is to define a common interface for your module. The
internal structure can be whatever form that you might like, but then
this code will all get re-exported in `mod.rs` to the rest of the world.

This also serves a convenience purpose: users of your module only have
to remember the module name, and you can keep whatever internal
structure is required.

For example, say we had the following folder structure:

```
myio/mod.rs
    /mem.rs
    /terminal/mod.rs
```

where we wish to keep `mem.rs` hidden from the outside world, and make
usage of `terminal` an explicit submodule. In `myio/mod.rs` we would
write:

```rust
// myio/mod.rs

pub use self::mem::MemReader;

mod mem;
pub mod terminal;
```

### Export common traits, structs, and enums at the module level

> **[OPEN]**

In the above example, we re-export `MemReader`, but we might have others
that are common to the whole module, and not just `mem.rs`:

```rust
// myio/mod.rs

pub enum FileMode { /* ... */ }
pub trait Seek { /* ... */ }
pub struct File { /* ... */ }
```

Then, to use these common traits in submodules:

```rust
// myio/mem.rs

use super::Seek;

pub struct MemReader { /* ... */ }
impl MemReader { /* ... */ }
impl Seek for MemReader { /* ... */ }
```

Notice how both `Seek` and `MemReader` are both visible from
`myio::Seek` and `myio::MemReader`.

### Use private modules to hide information

> **[OPEN]**

This structure lets you achieve the goals of information hiding (the
implementation of `mem` is separate from the `MemReader` in our API) and
making all useful types available for the internal modules.

It is good practice to keep code that is likely to change hidden in this
manner, and only make public the parts that constitute the module's
interface.
