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
/// # fn main() -> Result<(), Box<dyn Error>> {
/// your;
/// example?;
/// code;
/// #
/// #     Ok(())
/// # }
/// ```
```


<a id="c-failure"></a>
## Function docs include error, panic, and safety considerations (C-FAILURE)

Error conditions should be documented in an "Errors" section. This applies to
trait methods as well -- trait methods for which the implementation is allowed
or expected to return an error should be documented with an "Errors" section.

For example in the standard library, Some implementations of the
[`std::io::Read::read`] trait method may return an error.

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

Panic conditions should be documented in a "Panics" section. This applies to
trait methods as well -- traits methods for which the implementation is allowed
or expected to panic should be documented with a "Panics" section.

In the standard library the [`Vec::insert`] method may panic.

[`Vec::insert`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.insert

```
/// Inserts an element at position `index` within the vector, shifting all
/// elements after it to the right.
///
/// # Panics
///
/// Panics if `index` is out of bounds.
```

It is not necessary to document all conceivable panic cases, especially if the
panic occurs in logic provided by the caller. For example documenting the
`Display` panic in the following code seems excessive. But when in doubt, err on
the side of documenting more panic cases.

```rust
/// # Panics
///
/// This function panics if `T`'s implementation of `Display` panics.
pub fn print<T: Display>(t: T) {
    println!("{}", t.to_string());
}
```

Unsafe functions should be documented with a "Safety" section that explains all
invariants that the caller is responsible for upholding to use the function
correctly.

The unsafe [`std::ptr::read`] requires the following of the caller.

[`std::ptr::read`]: https://doc.rust-lang.org/std/ptr/fn.read.html

```
/// Reads the value from `src` without moving it. This leaves the
/// memory in `src` unchanged.
///
/// # Safety
///
/// Beyond accepting a raw pointer, this is unsafe because it semantically
/// moves the value out of `src` without preventing further usage of `src`.
/// If `T` is not `Copy`, then care must be taken to ensure that the value at
/// `src` is not used before the data is overwritten again (e.g. with `write`,
/// `zero_memory`, or `copy_memory`). Note that `*src = foo` counts as a use
/// because it will attempt to drop the value previously at `*src`.
///
/// The pointer must be aligned; use `read_unaligned` if that is not the case.
```


<a id="c-link"></a>
## Prose contains hyperlinks to relevant things (C-LINK)

Regular links can be added inline with the usual markdown syntax of
`[text](url)`. Links to other types can be added by marking them with
``[`text`]``, then adding the link target in a new line at the end of
the docstring with ``[`text`]: <target>``, where `<target>` is
described below.

Link targets to methods within the same type usually look like this:

```md
[`serialize_struct`]: #method.serialize_struct
```

Link targets to other types usually look like this:

```md
[`Deserialize`]: trait.Deserialize.html
```

Link targets may also point to a parent or child module:

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
- `repository`
- `readme`
- `keywords`
- `categories`

In addition, there are two optional metadata fields:

- `documentation`
- `homepage`

By default, *crates.io* links to documentation for the crate on [*docs.rs*]. The
`documentation` metadata only needs to be set if the documentation is hosted
somewhere other than *docs.rs*, for example because the crate links against a
shared library that is not available in the build environment of *docs.rs*.

[*docs.rs*]: https://docs.rs

The `homepage` metadata should only be set if there is a unique website for the
crate other than the source repository or API documentation. Do not make
`homepage` redundant with either the `documentation` or `repository` values. For
example, serde sets `homepage` to *https://serde.rs*, a dedicated website.

<a id="c-relnotes"></a>
## Release notes document all significant changes (C-RELNOTES)

Users of the crate can read the release notes to find a summary of what
changed in each published release of the crate. A link to the release notes,
or the notes themselves, should be included in the crate-level documentation
and/or the repository linked in Cargo.toml.

Breaking changes (as defined in [RFC 1105]) should be clearly identified in the
release notes.

If using Git to track the source of a crate, every release published to
*crates.io* should have a corresponding tag identifying the commit that was
published. A similar process should be used for non-Git VCS tools as well.

```bash
# Tag the current commit
GIT_COMMITTER_DATE=$(git log -n1 --pretty=%aD) git tag -a -m "Release 0.3.0" 0.3.0
git push --tags
```

Annotated tags are preferred because some Git commands ignore unannotated tags
if any annotated tags exist.

[RFC 1105]: https://github.com/rust-lang/rfcs/blob/master/text/1105-api-evolution.md

### Examples

- [Serde 1.0.0 release notes](https://github.com/serde-rs/serde/releases/tag/v1.0.0)
- [Serde 0.9.8 release notes](https://github.com/serde-rs/serde/releases/tag/v0.9.8)
- [Serde 0.9.0 release notes](https://github.com/serde-rs/serde/releases/tag/v0.9.0)
- [Diesel change log](https://github.com/diesel-rs/diesel/blob/master/CHANGELOG.md)


<a id="c-hidden"></a>
## Rustdoc does not show unhelpful implementation details (C-HIDDEN)

Rustdoc is supposed to include everything users need to use the crate fully and
nothing more. It is fine to explain relevant implementation details in prose but
they should not be real entries in the documentation.

Especially be selective about which impls are visible in rustdoc -- all the ones
that users would need for using the crate fully, but no others. In the following
code the rustdoc of `PublicError` by default would show the `From<PrivateError>`
impl. We choose to hide it with `#[doc(hidden)]` because users can never have a
`PrivateError` in their code so this impl would never be relevant to them.

```rust
// This error type is returned to users.
pub struct PublicError { /* ... */ }

// This error type is returned by some private helper functions.
struct PrivateError { /* ... */ }

// Enable use of `?` operator.
#[doc(hidden)]
impl From<PrivateError> for PublicError {
    fn from(err: PrivateError) -> PublicError {
        /* ... */
    }
}
```

[`pub(crate)`] is another great tool for removing implementation details from
the public API. It allows items to be used from outside of their own module but
not outside of the same crate.

[`pub(crate)`]: https://github.com/rust-lang/rfcs/blob/master/text/1422-pub-restricted.md
