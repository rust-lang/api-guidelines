# Future proofing


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

In the future the same thing can be accomplished more concisely with the [`impl
Trait`] feature but this is currently unstable.

[`impl Trait`]: https://github.com/rust-lang/rfcs/blob/master/text/1522-conservative-impl-trait.md

```rust
#![feature(conservative_impl_trait)]

pub fn my_transform<I: Iterator>(input: I) -> impl Iterator<Item = (usize, I::Item)> {
    input.skip(3).enumerate()
}
```


<a id="c-struct-bounds"></a>
## Data structures do not duplicate derived trait bounds (C-STRUCT-BOUNDS)

Generic data structures should not use trait bounds that can be derived or don't otherwise 
add semantic value.
Each trait in the `derive` attribute will be expanded into a separate `impl` block that
only applies to generic arguments that implement that trait.

```rust
// Prefer this:
#[derive(Clone, Debug, PartialEq)]
struct Good<T> { /* ... */ }

// Over this:
#[derive(Clone, Debug, PartialEq)]
struct Bad<T: Clone + Debug + PartialEq> { /* ... */ }
```

Duplicating derived traits as bounds on `Bad` is unnecessary and a 
backwards-compatibiliity hazard.
To illustrate this point, consider deriving `PartialOrd` on the structures in the 
previous example:

```rust
// Non-breaking change:
#[derive(Clone, Debug, PartialEq, PartialOrd)]
struct Good<T> { /* ... */ }

// Breaking change:
#[derive(Clone, Debug, PartialEq, PartialOrd)]
struct Bad<T: Clone + Debug + PartialEq + PartialOrd> { /* ... */ }
```

Generally speaking, adding a trait bound to a data structure is a breaking change 
because every consumer of that structure will need to start satisfying the additional bound.
Deriving more traits from the standard library using the `derive` attribute is not a 
breaking change.

The following traits should always be avoided in bounds on data structures:

- `Clone`
- `PartialEq`
- `PartialOrd`
- `Debug`
- `Display`
- `Default`
- `Serialize`
- `Deserialize`
- `DeserializeOwned`

There's a grey area around other non-derivable trait bounds that aren't strictly 
required by the structure definition, like `Read` or `Write`.
They may communicate the intented behaviour of the type better in its definition 
but also limits future extensibility.
Including semantically useful trait bounds on data structures is still less 
problematic than including derivable traits as bounds.

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
