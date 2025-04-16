# Future proofing


<a id="c-sealed"></a>
## Sealed traits protect against downstream implementations (C-SEALED)

Some traits are only meant to be implemented within the crate that defines them.
In such cases, we can retain the ability to make changes to the trait in a
non-breaking way by using the sealed trait pattern.

```rust
/// This trait is sealed and cannot be implemented for types outside this crate.
pub trait TheTrait: private::Sealed {
    // Zero or more methods that the user is allowed to call.
    fn ...();

    // Zero or more private methods, not allowed for user to call.
    #[doc(hidden)]
    fn ...();
}

// Implement for some types.
impl TheTrait for usize {
    /* ... */
}

mod private {
    pub trait Sealed {}

    // Implement for those same types, but no others.
    impl Sealed for usize {}
}
```

The empty private `Sealed` supertrait cannot be named by downstream crates, so
we are guaranteed that implementations of `Sealed` (and therefore `TheTrait`)
only exist in the current crate. We are free to add methods to `TheTrait` in a
non-breaking release even though that would ordinarily be a breaking change for
traits that are not sealed. Also we are free to change the signature of methods
that are not publicly documented.

Note that removing a public method or changing the signature of a public method
in a sealed trait are still breaking changes.

To avoid frustrated users trying to implement the trait, it should be documented
in rustdoc that the trait is sealed and not meant to be implemented outside of
the current crate.

### Examples

- [`serde_json::value::Index`](https://docs.serde.rs/serde_json/value/trait.Index.html)
- [`byteorder::ByteOrder`](https://docs.rs/byteorder/1.1.0/byteorder/trait.ByteOrder.html)


<a id="c-struct-private"></a>
## Structs have private fields (C-STRUCT-PRIVATE)

Making a field public is a strong commitment: it pins down a representation
choice, _and_ prevents the type from providing any validation or maintaining any
invariants on the contents of the field, since clients can mutate it arbitrarily.

Public fields are most appropriate for `struct` types in the C spirit: compound,
passive data structures. Otherwise, consider providing getter/setter methods and
hiding fields instead.


<a id="c-newtype-hide"></a>
## Newtypes encapsulate implementation details (C-NEWTYPE-HIDE)

A newtype can be used to hide representation details while making precise
promises to the client.

For example, consider a function `my_transform` that returns a compound iterator
type.

```rust
use std::iter::{Enumerate, Skip};

pub fn my_transform<I: Iterator>(input: I) -> Enumerate<Skip<I>> {
    input.skip(3).enumerate()
}
```

We wish to hide this type from the client, so that the client's view of the
return type is roughly `Iterator<Item = (usize, T)>`. We can do so using the
newtype pattern:

```rust
use std::iter::{Enumerate, Skip};

pub struct MyTransformResult<I>(Enumerate<Skip<I>>);

impl<I: Iterator> Iterator for MyTransformResult<I> {
    type Item = (usize, I::Item);

    fn next(&mut self) -> Option<Self::Item> {
        self.0.next()
    }
}

pub fn my_transform<I: Iterator>(input: I) -> MyTransformResult<I> {
    MyTransformResult(input.skip(3).enumerate())
}
```

Aside from simplifying the signature, this use of newtypes allows us to promise
less to the client. The client does not know _how_ the result iterator is
constructed or represented, which means the representation can change in the
future without breaking client code.

Rust 1.26 also introduces the [`impl Trait`][] feature, which is more concise
than the newtype pattern but with some additional trade offs, namely with `impl
Trait` you are limited in what you can express.  For example, returning an
iterator that impls `Debug` or `Clone` or some combination of the other iterator
extension traits can be problematic.  In summary `impl Trait` as a return type
is probably great for internal APIs and may even be appropriate for public APIs,
but probably not in all cases.  See the ["`impl Trait` for returning complex
types with ease"][impl-trait-2] section of the Edition Guide for more details.

[`impl Trait`]: https://github.com/rust-lang/rfcs/blob/master/text/1522-conservative-impl-trait.md
[impl-trait-2]: https://rust-lang.github.io/edition-guide/rust-2018/trait-system/impl-trait-for-returning-complex-types-with-ease.html

```rust
pub fn my_transform<I: Iterator>(input: I) -> impl Iterator<Item = (usize, I::Item)> {
    input.skip(3).enumerate()
}
```


<a id="c-struct-bounds"></a>
## Data structures do not duplicate derived trait bounds (C-STRUCT-BOUNDS)

Generic data structures should not use trait bounds that can be derived or do
not otherwise add semantic value. Each trait in the `derive` attribute will be
expanded into a separate `impl` block that only applies to generic arguments
that implement that trait.

```rust
// Prefer this:
#[derive(Clone, Debug, PartialEq)]
struct Good<T> { /* ... */ }

// Over this:
#[derive(Clone, Debug, PartialEq)]
struct Bad<T: Clone + Debug + PartialEq> { /* ... */ }
```

Duplicating derived traits as bounds on `Bad` is unnecessary and a
backwards-compatibiliity hazard. To illustrate this point, consider deriving
`PartialOrd` on the structures in the previous example:

```rust
// Non-breaking change:
#[derive(Clone, Debug, PartialEq, PartialOrd)]
struct Good<T> { /* ... */ }

// Breaking change:
#[derive(Clone, Debug, PartialEq, PartialOrd)]
struct Bad<T: Clone + Debug + PartialEq + PartialOrd> { /* ... */ }
```

Generally speaking, adding a trait bound to a data structure is a breaking
change because every consumer of that structure will need to start satisfying
the additional bound. Deriving more traits from the standard library using the
`derive` attribute is not a breaking change.

The following traits should never be used in bounds on data structures:

- `Clone`
- `PartialEq`
- `PartialOrd`
- `Debug`
- `Display`
- `Default`
- `Error`
- `Serialize`
- `Deserialize`
- `DeserializeOwned`

There is a grey area around other non-derivable trait bounds that are not
strictly required by the structure definition, like `Read` or `Write`. They may
communicate the intended behavior of the type better in its definition but also
limits future extensibility. Including semantically useful trait bounds on data
structures is still less problematic than including derivable traits as bounds.

### Exceptions

There are three exceptions where trait bounds on structures are required:

1. The data structure refers to an associated type on the trait.
1. The bound is `?Sized`.
1. The data structure has a `Drop` impl that requires trait bounds.
Rust currently requires all trait bounds on the `Drop` impl are also present
on the data structure.

### Examples from the standard library

- [`std::borrow::Cow`] refers to an associated type on the `Borrow` trait.
- [`std::boxed::Box`] opts out of the implicit `Sized` bound.
- [`std::io::BufWriter`] requires a trait bound in its `Drop` impl.

[`std::borrow::Cow`]: https://doc.rust-lang.org/std/borrow/enum.Cow.html
[`std::boxed::Box`]: https://doc.rust-lang.org/std/boxed/struct.Box.html
[`std::io::BufWriter`]: https://doc.rust-lang.org/std/io/struct.BufWriter.html

<a id="c-nightly-optin"></a>
## Nightly features use an explicit opt-in (C-NIGHTLY-OPTIN)

Some libraries need to use, or want to experiment with, the [nightly channel].
To avoid accidental breakage, libraries should either:
- Use nightly features unconditionally, so that people depending on the library must always use a nightly toolchain to build
- Add a `--cfg` flag which opts-in to the nightly features (optionally, with feature detection to verify the features are present in the current compiler version). This allows people to avoid opting-in if they do not want to be exposed to possible breakage.

Each nightly feature should be under a separate flag so that breakage to one feature does not cause breakage for others.
Each flag should be prefixed with the crate name to avoid accidentally triggering features in other crates.
For example, if your crate is named `awesome-lib` and depends on two different nightly features, `std::intrinsics::black_box` and `std::intrinsics::catch_unwind`, create two cfgs named `awesome_lib_black_box` and `awesome_lib_catch_unwind`.

When doing feature detection, we recommend *against* simply checking whether the compiler is a nightly channel, as nightly features frequently change between compiler versions.
Feature detection should compile a sample rust program and verify that it works.
Note that flags like `-Z allow-features` may affect whether features are allowed.

[nightly channel]: https://rust-lang.github.io/rustup/concepts/channels.html
