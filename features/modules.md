# Modules

> **[OPEN]** What general guidelines should we provide for module design?

> We should discuss visibility, nesting, `mod.rs`, and any interesting patterns
> around modules.

#### Naming conventions
> **[OPEN]**
> - Anything else?
> - Are there cases where *not* separating words with underscores is OK,
>   or should this be a hard rule?

- Module names should contain only lowercae letters and underscores.
  For example, use `std::io::timer`, not `Std::IO::Timer`.
- Multiple words should be separated by underscores.
  Use `std::local_data`, not `std::localData` or `std::localdata`.

#### Headers
> **[OPEN]** Is this header organization suggestion valid?

Organize module headers as follows:
  1. [Imports](../style/imports.md).
  1. `mod` declarations.
  1. `pub mod` declarations.

#### Avoid `path` directives
> **[OPEN]** This is hardly ever seen in the Rust codebase (only 4 uses, all in
> `libsyntax`) and seems like overall a bad idea.

Avoid using `#[path="..."]` directives except where it is *absolutely*
  necessary.

### Use the module hirearchy to organize APIs into coherent sections
> **[OPEN]**

The module hirearchy defines both the public and internal API of your module.
Breaking related functionality into submodules makes it understandable to both
users and contributors to the module.

#### Place modules in separate files
> **[OPEN]**
> - "<100 lines" is completely arbitrary, but it's a clearer recommendation
>   than "~1 page" or similar suggestions that vary by screen size, etc.

For all except very short modules (<100 lines) and [tests](../testing/README.md),
place the module `foo` in a separate file: either `foo.rs` or `foo/mod.rs`,
depending on your needs, rather than declaring it inline like

```rust
pub mod foo {
    pub fn bar() { println!("..."); }
    /* ... */
}
```

#### Use folders to organize submodules
> **[OPEN]**

For modules that themselves have submodules, place the module in a separate
folder (e.g., `bar/mod.rs` for a module `bar`) rather than the same directory.

Note the structure of
[`std::io`](http://doc.rust-lang.org/std/io/). Many of the submodules lack
children, like
[`io::fs`](http://doc.rust-lang.org/std/io/fs/)
and
[`io::stdio`](http://doc.rust-lang.org/std/io/stdio/).
On the other hand,
[`io::net`](http://doc.rust-lang.org/std/io/net/)
contains submodules, so it lives in a separate folder:

```
io/mod.rs
   io/extensions.rs
   io/fs.rs
   io/net/mod.rs
          io/net/addrinfo.rs
          io/net/ip.rs
          io/net/tcp.rs
          io/net/udp.rs
          io/net/unix.rs
   io/pipe.rs
   ...
```

While it is possible to define all of `io` within a single folder, mirroring
the module hirearchy in the directory structure makes submodules of `io::net`
easier to find.

#### Top-level definitions
> **[OPEN]**

Define or [reexport](http://doc.rust-lang.org/std/io/#reexports) commonly used
definitions at the top level of your module.

Functionality that is related to the module itself should be defined in
`mod.rs`, while functionality specific to a submodule should live in its
related submodule and be reexported elsewhere.

For example,
[`IoError`](http://doc.rust-lang.org/std/io/struct.IoError.html)
is defined in `io/mod.rs`, since it pertains to the entirety of the submodule,
while
[`TcpStream`](http://doc.rust-lang.org/std/io/net/tcp/struct.TcpStream.html)
is defined in `io/net/tcp.rs` and reexported in the `io` module.

### Use internal module hirearchies for hiding implementations
> **[OPEN]**
> - Referencing internal modules from the standard library is subject to
>   becoming outdated.

Internal module hirearchies (including private submodules) may be used to
hide implementation details that are not part of the module's API.

For example, in [`std::io`](http://doc.rust-lang.org/std/io/), `mod mem`
provides implementations for
[`BufReader`](http://doc.rust-lang.org/std/io/struct.BufReader.html)
and
[`BufWriter`](http://doc.rust-lang.org/std/io/struct.BufWriter.html),
but these are re-exported in `io/mod.rs` at the top level of the module:

```rust
// libstd/io/mod.rs

pub use self::mem::{MemReader, BufReader, MemWriter, BufWriter};
/* ... */
mod mem;
```

This hides the detail that there even exists a `mod mem` in `io`, and
helps keep code organized while offering freedom to change the implementation.
