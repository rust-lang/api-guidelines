# Documentation


<a id="c-crate-doc"></a>
## Crate level docs are thorough and include examples (C-CRATE-DOC)

See [RFC 1687].

[RFC 1687]: https://github.com/rust-lang/rfcs/pull/1687


<a id="c-example"></a>
## All items have a rustdoc example (C-EXAMPLE)

Every public module, trait, struct, enum, function, method, macro, and type
definition should have an example that exercises the functionality.

This guideline should be applied within reason.

A link to an applicable example on another item may be sufficient. For example
if exactly one function uses a particular type, it may be appropriate to write a
single example on either the function or the type and link to it from the other.

The purpose of an example is not always to show *how to use* the item. Readers
can be expected to understand how to invoke functions, match on enums, and other
fundamental tasks. Rather, an example is often intended to show *why someone
would want to use* the item.

```rust
// This would be a poor example of using clone(). It mechanically shows *how* to
// call clone(), but does nothing to show *why* somebody would want this.
fn main() {
    let hello = "hello";

    hello.clone();
}
```


<a id="c-question-mark"></a>
## Examples use `?`, not `try!`, not `unwrap` (C-QUESTION-MARK)

Like it or not, example code is often copied verbatim by users. Unwrapping an
error should be a conscious decision that the user needs to make.

A common way of structuring fallible example code is the following. The lines
beginning with `#` are compiled by `cargo test` when building the example but
will not appear in user-visible rustdoc.

```
/// ```rust
/// # use std::error::Error;
/// #
/// # fn try_main() -> Result<(), Box<Error>> {
/// your;
/// example?;
/// code;
/// #
/// #     Ok(())
/// # }
/// #
/// # fn main() {
/// #     try_main().unwrap();
/// # }
/// ```
```


<a id="c-error-doc"></a>
## Function docs include error conditions in "Errors" section (C-ERROR-DOC)

Per [RFC 1574].

This applies to trait methods as well. Trait methods for which the
implementation is allowed or expected to return an error should be documented
with an "Errors" section.

[RFC 1574]: https://github.com/rust-lang/rfcs/blob/master/text/1574-more-api-documentation-conventions.md

### Examples from the standard library

Some implementations of the [`std::io::Read::read`] trait method may return an
error.

[`std::io::Read::read`]: https://doc.rust-lang.org/std/io/trait.Read.html#tymethod.read

```
/// Pull some bytes from this source into the specified buffer, returning
/// how many bytes were read.
///
/// ... lots more info ...
///
/// # Errors
///
/// If this function encounters any form of I/O or other error, an error
/// variant will be returned. If an error is returned then it must be
/// guaranteed that no bytes were read.
```


<a id="c-panic-doc"></a>
## Function docs include panic conditions in "Panics" section (C-PANIC-DOC)

Per [RFC 1574].

This applies to trait methods as well. Traits methods for which the
implementation is allowed or expected to panic should be documented with a
"Panics" section.

### Examples from the standard library

The [`Vec::insert`] method may panic.

[`Vec::insert`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.insert

```
/// Inserts an element at position `index` within the vector, shifting all
/// elements after it to the right.
///
/// # Panics
///
/// Panics if `index` is out of bounds.
```


<a id="c-link"></a>
## Prose contains hyperlinks to relevant things (C-LINK)

Links to methods within the same type usually look like this:

```md
[`serialize_struct`]: #method.serialize_struct
```

Links to other types usually look like this:

```md
[`Deserialize`]: trait.Deserialize.html
```

Links may also point to a parent or child module:

```md
[`Value`]: ../enum.Value.html
[`DeserializeOwned`]: de/trait.DeserializeOwned.html
```

This guideline is officially recommended by RFC 1574 under the heading ["Link
all the things"].

["Link all the things"]: https://github.com/rust-lang/rfcs/blob/master/text/1574-more-api-documentation-conventions.md#link-all-the-things


<a id="c-metadata"></a>
## Cargo.toml includes all common metadata (C-METADATA)

The `[package]` section of `Cargo.toml` should include the following
values:

- `authors`
- `description`
- `license`
- `documentation`
- `repository`
- `readme`
- `keywords`
- `categories`

`Cargo.toml` also allows for a `homepage` value. This should only be
filled in if there is a unique website for the crate other than the
source repository or API documentation. Do not make `homepage`
redundant with either the `documentation` or `repository` values. For
example, serde sets `homepage` to "https://serde.rs", a dedicated
website.


<a id="c-docs-rs"></a>
## Cargo.toml documentation key points to docs.rs (C-DOCS-RS)

In most cases the `documentation` key in a crate's `Cargo.toml` should
be `"https://docs.rs/CRATE"`

```toml
[package]
name = "regex"
version = "0.2.2"
documentation = "https://docs.rs/regex"
```

[docs.rs] provides automatic hosting of crate documentation, with a
consistent style, inter-crate navigation, and advanced features not
included in the stock rustdoc output. It is a central destination for
documentation of all Rust crates. Rust users are familiar with docs.rs
and expect to go there for their API documentation. A crate that
directs their users to a different site for its documentation will be
noticably segregating itself from the larger portion of the
ecosystem. This may be desirable to crate authors in some situations
but should be considered carefully.

[docs.rs]: https://docs.rs


<a id="c-relnotes"></a>
## Release notes document all significant changes (C-RELNOTES)

Users of the crate can read the release notes to find a summary of what
changed in each published release of the crate.  A link to the release notes,
or the notes themselves, should be included in the crate-level documentation
and/or the repository linked in Cargo.toml.

Breaking changes (as defined in [RFC 1105]) should be clearly identified in the
release notes.

[RFC 1105]: https://github.com/rust-lang/rfcs/blob/master/text/1105-api-evolution.md

### Examples

- [Serde 1.0.0 release notes](https://github.com/serde-rs/serde/releases/tag/v1.0.0)
- [Serde 0.9.8 release notes](https://github.com/serde-rs/serde/releases/tag/v0.9.8)
- [Serde 0.9.0 release notes](https://github.com/serde-rs/serde/releases/tag/v0.9.0)
- [Diesel change log](https://github.com/diesel-rs/diesel/blob/master/CHANGELOG.md)
