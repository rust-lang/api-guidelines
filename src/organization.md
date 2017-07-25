# Organization


<a id="c-reexport"></a>
## Crate root re-exports common functionality (C-REEXPORT)

Crates `pub use` the most common types for convenience, so that clients do not
have to remember or write the crate's module hierarchy to use these types.

Re-exporting is covered in more detail in the *The Rust Programming Language*
under [Crates and Modules][reexport].

[reexport]: https://doc.rust-lang.org/book/first-edition/crates-and-modules.html#re-exporting-with-pub-use

### Examples from `serde_json`

The [`serde_json::Value`] type is the most commonly used type from `serde_json`.
It is a re-export of a type that lives elsewhere in the module hierarchy, at
`serde_json::value::Value`. The [`serde_json::value`][value-mod] module defines
other JSON-value-related things that are not re-exported. For example
[`serde_json::value::Index`] is the trait that defines types that can be used to
index into a `Value` using square bracket indexing notation. The `Index` trait
is not re-exported at the crate root because it would be comparatively rare for
a client crate to need to refer to it.

[`serde_json::Value`]: https://docs.serde.rs/serde_json/enum.Value.html
[value-mod]: https://docs.serde.rs/serde_json/value/index.html
[`serde_json::value::Index`]: https://docs.serde.rs/serde_json/value/trait.Index.html

In addition to types, functions can be re-exported as well. In `serde_json` the
[`serde_json::from_str`] function is a re-export of a function from the
[`serde_json::de`] deserialization module, which contains other less common
deserialization-related functionality that is not re-exported.

[`serde_json::from_str`]: https://docs.serde.rs/serde_json/fn.from_str.html
[`serde_json::de`]: https://docs.serde.rs/serde_json/de/index.html


<a id="c-hierarchy"></a>
## Modules provide a sensible API hierarchy (C-HIERARCHY)

### Examples from Serde

The `serde` crate is two independent frameworks in one crate - a serialization
half and a deserialization half. The crate is divided accordingly into
[`serde::ser`] and [`serde::de`]. Part of the deserialization framework is
isolated under [`serde::de::value`] because it is a relatively large API surface
that is relatively unimportant, and it would crowd the more common, more
important functionality located in `serde::de` if it were to share the same
namespace.

[`serde::ser`]: https://docs.serde.rs/serde/ser/index.html
[`serde::de`]: https://docs.serde.rs/serde/de/index.html
[`serde::de::value`]: https://docs.serde.rs/serde/de/value/index.html
