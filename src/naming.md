# Naming


<a id="c-case"></a>
## Casing conforms to RFC 430 (C-CASE)

Basic Rust naming conventions are described in [RFC 430].

In general, Rust tends to use `CamelCase` for "type-level" constructs (types and
traits) and `snake_case` for "value-level" constructs. More precisely:

| Item | Convention |
| ---- | ---------- |
| Crates | [unclear](https://github.com/rust-lang-nursery/api-guidelines/issues/29) |
| Modules | `snake_case` |
| Types | `CamelCase` |
| Traits | `CamelCase` |
| Enum variants | `CamelCase` |
| Functions | `snake_case` |
| Methods | `snake_case` |
| General constructors | `new` or `with_more_details` |
| Conversion constructors | `from_some_other_type` |
| Local variables | `snake_case` |
| Static variables | `SCREAMING_SNAKE_CASE` |
| Constant variables | `SCREAMING_SNAKE_CASE` |
| Type parameters | concise `CamelCase`, usually single uppercase letter: `T` |
| Lifetimes | short `lowercase`, usually a single letter: `'a`, `'de`, `'src` |
| Features | [unclear](https://github.com/rust-lang-nursery/api-guidelines/issues/101) but see [C-FEATURE] |

In `CamelCase`, acronyms count as one word: use `Uuid` rather than `UUID`. In
`snake_case`, acronyms are lower-cased: `is_xid_start`.

In `snake_case` or `SCREAMING_SNAKE_CASE`, a "word" should never consist of a
single letter unless it is the last "word". So, we have `btree_map` rather than
`b_tree_map`, but `PI_2` rather than `PI2`.

[RFC 430]: https://github.com/rust-lang/rfcs/blob/master/text/0430-finalizing-naming-conventions.md
[C-FEATURE]: #c-feature

### Examples from the standard library

The whole standard library. This guideline should be easy!


<a id="c-conv"></a>
## Ad-hoc conversions follow `as_`, `to_`, `into_` conventions (C-CONV)

Conversions should be provided as methods, with names prefixed as follows:

| Prefix | Cost | Ownership |
| ------ | ---- | --------- |
| `as_` | Free | borrowed -\> borrowed |
| `to_` | Expensive | borrowed -\> owned (non-Copy types)<br>owned -\> owned (Copy types) |
| `into_` | Variable | owned -\> owned (non-Copy types) |

For example:

- [`str::as_bytes()`] gives a view of a `str` as a slice of UTF-8 bytes, which
  is free. The input is a borrowed `&str` and the output is a borrowed `&[u8]`.
- [`str::to_lowercase()`] produces the Unicode-correct lowercase equivalent of a
  `str`, which involves iterating through characters of the string and may
  require memory allocation. The input is a borrowed `&str` and the output is an
  owned `String`.
- [`f64::to_radians()`] converts a floating point quantity from degrees to
  radians. The input is `f64`. Passing a reference `&f64` is not warranted
  because `f64` is cheap to copy. Calling the function `into_radians` would be
  misleading because the input is not consumed.
- [`String::into_bytes()`] extracts the underlying `Vec<u8>` of a `String`,
  which is free. It takes ownership of a `String` and returns an owned
  `Vec<u8>`.
- [`BufReader::into_inner()`] takes ownership of a buffered reader and extracts
  out the underlying reader, which is free. Data in the buffer is discarded.
- [`BufWriter::into_inner()`] takes ownership of a buffered writer and extracts
  out the underlying writer, which requires a potentially expensive flush of any
  buffered data.

[`str::as_bytes()`]: https://doc.rust-lang.org/std/primitive.str.html#method.as_bytes
[`str::to_lowercase()`]: https://doc.rust-lang.org/std/primitive.str.html#method.to_lowercase
[`f64::to_radians()`]: https://doc.rust-lang.org/std/primitive.f64.html#method.to_radians
[`String::into_bytes()`]: https://doc.rust-lang.org/std/string/struct.String.html#method.into_bytes
[`BufReader::into_inner()`]: https://doc.rust-lang.org/std/io/struct.BufReader.html#method.into_inner
[`BufWriter::into_inner()`]: https://doc.rust-lang.org/std/io/struct.BufWriter.html#method.into_inner

Conversions prefixed `as_` and `into_` typically _decrease abstraction_, either
exposing a view into the underlying representation (`as`) or deconstructing data
into its underlying representation (`into`). Conversions prefixed `to_`, on the
other hand, typically stay at the same level of abstraction but do some work to
change one representation into another.

##### More examples from the standard library

- [`Result::as_ref`](https://doc.rust-lang.org/std/result/enum.Result.html#method.as_ref)
- [`RefCell::as_ptr`](https://doc.rust-lang.org/std/cell/struct.RefCell.html#method.as_ptr)
- [`Path::to_str`](https://doc.rust-lang.org/std/path/struct.Path.html#method.to_str)
- [`slice::to_vec`](https://doc.rust-lang.org/std/primitive.slice.html#method.to_vec)
- [`Option::into_iter`](https://doc.rust-lang.org/std/option/enum.Option.html#method.into_iter)
- [`AtomicBool::into_inner`](https://doc.rust-lang.org/std/sync/atomic/struct.AtomicBool.html#method.into_inner)


<a id="c-iter"></a>
## Methods on collections that produce iterators follow `iter`, `iter_mut`, `into_iter` (C-ITER)

Per [RFC 199].

For a container with elements of type `U`, iterator methods should be named:

```rust
fn iter(&self) -> Iter             // Iter implements Iterator<Item = &U>
fn iter_mut(&mut self) -> IterMut  // IterMut implements Iterator<Item = &mut U>
fn into_iter(self) -> IntoIter     // IntoIter implements Iterator<Item = U>
```

This guideline applies to data structures that are conceptually homogeneous
collections. As a counterexample, the `str` type is slice of bytes that are
guaranteed to be valid UTF-8. This is conceptually more nuanced than a
homogeneous collection so rather than providing the
`iter`/`iter_mut`/`into_iter` group of iterator methods, it provides
[`str::bytes`] to iterate as bytes and [`str::chars`] to iterate as chars.

[`str::bytes`]: https://doc.rust-lang.org/std/primitive.str.html#method.bytes
[`str::chars`]: https://doc.rust-lang.org/std/primitive.str.html#method.chars

This guideline applies to methods only, not functions. For example
[`percent_encode`] from the `url` crate returns an iterator over percent-encoded
string fragments. There would be no clarity to be had by using an
`iter`/`iter_mut`/`into_iter` convention.

[`percent_encode`]: https://docs.rs/url/1.4.0/url/percent_encoding/fn.percent_encode.html
[RFC 199]: https://github.com/rust-lang/rfcs/blob/master/text/0199-ownership-variants.md

### Examples from the standard library

- [`Vec::iter`](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.iter)
- [`Vec::iter_mut`](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.iter_mut)
- [`Vec::into_iter`](https://doc.rust-lang.org/std/vec/struct.Vec.html#method.into_iter)
- [`BTreeMap::iter`](https://doc.rust-lang.org/std/collections/struct.BTreeMap.html#method.iter)
- [`BTreeMap::iter_mut`](https://doc.rust-lang.org/std/collections/struct.BTreeMap.html#method.iter_mut)


<a id="c-iter-ty"></a>
## Iterator type names match the methods that produce them (C-ITER-TY)

A method called `into_iter()` should return a type called `IntoIter` and
similarly for all other methods that return iterators.

This guideline applies chiefly to methods, but often makes sense for functions
as well. For example the [`percent_encode`] function from the `url` crate
returns an iterator type called [`PercentEncode`][PercentEncode-type].

[PercentEncode-type]: https://docs.rs/url/1.4.0/url/percent_encoding/struct.PercentEncode.html

These type names make the most sense when prefixed with their owning module, for
example [`vec::IntoIter`].

[`vec::IntoIter`]: https://doc.rust-lang.org/std/vec/struct.IntoIter.html

### Examples from the standard library

* [`Vec::iter`] returns [`Iter`][slice::Iter]
* [`Vec::iter_mut`] returns [`IterMut`][slice::IterMut]
* [`Vec::into_iter`] returns [`IntoIter`][vec::IntoIter]
* [`BTreeMap::keys`] returns [`Keys`][btree_map::Keys]
* [`BTreeMap::values`] returns [`Values`][btree_map::Values]

[`Vec::iter`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.iter
[slice::Iter]: https://doc.rust-lang.org/std/slice/struct.Iter.html
[`Vec::iter_mut`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.iter_mut
[slice::IterMut]: https://doc.rust-lang.org/std/slice/struct.IterMut.html
[`Vec::into_iter`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.into_iter
[vec::IntoIter]: https://doc.rust-lang.org/std/vec/struct.IntoIter.html
[`BTreeMap::keys`]: https://doc.rust-lang.org/std/collections/struct.BTreeMap.html#method.keys
[btree_map::Keys]: https://doc.rust-lang.org/std/collections/btree_map/struct.Keys.html
[`BTreeMap::values`]: https://doc.rust-lang.org/std/collections/struct.BTreeMap.html#method.values
[btree_map::Values]: https://doc.rust-lang.org/std/collections/btree_map/struct.Values.html


<a id="c-own-suffix"></a>
## Ownership suffixes use `_mut`, `_ref` (C-OWN-SUFFIX)

Functions often come in multiple variants: immutably borrowed, mutably borrowed,
and owned.

The right default depends on the function in question. Variants should be marked
through suffixes.

### Exceptions

In the case of iterators, the moving variant should be understood as an `into`
conversion, `into_iter`.

For mutably borrowed variants, if the `mut` qualifier is part of a type name,
it should appear as it would appear in the type. For example
[`Vec::as_mut_slice`] returns a mut slice; it does what it says.

[`Vec::as_mut_slice`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.as_mut_slice

### Immutably borrowed by default

If `foo` uses/produces an immutable borrow by default, use a `_mut` suffix (e.g.
`foo_mut`) for the mutably borrowed variant.

#### Examples from the standard library

TODO [api-guidelines#37](https://github.com/rust-lang-nursery/api-guidelines/issues/37)

### Owned by default

If `foo` uses/produces owned data by default, use:

* The `_ref` suffix (e.g. `foo_ref`) for the immutably borrowed variant.
* The `_mut` suffix (e.g. `foo_mut`) for the mutably borrowed variant.

#### Examples from the standard library

- [`std::io::BufReader::get_ref`](https://doc.rust-lang.org/std/io/struct.BufReader.html#method.get_ref)
- [`std::io::BufReader::get_mut`](https://doc.rust-lang.org/std/io/struct.BufReader.html#method.get_mut)


<a id="c-getters"></a>
## Single-element containers implement appropriate getters (C-GETTERS)

Single-element containers where accessing the element cannot fail should
implement `get` and `get_mut`, with the following signatures.

```rust
fn get(&self) -> &V;
fn get_mut(&mut self) -> &mut V;
```

Single-element containers where the element is [`Copy`] (e.g. [`Cell`]-like
containers) should instead return the value directly, and not implement a
mutable accessor. TODO [api-guidelines#44]

[api-guidelines#44]: https://github.com/rust-lang-nursery/api-guidelines/issues/44

[`Copy`]: https://doc.rust-lang.org/std/marker/trait.Copy.html
[`Cell`]: https://doc.rust-lang.org/std/cell/struct.Cell.html

```rust
fn get(&self) -> V;
```

For getters that do runtime validation, consider adding unsafe `_unchecked`
variants.

```rust
unsafe fn get_unchecked(&self, index) -> &V;
```

### Examples from the standard library

- [`std::io::Cursor::get_mut`](https://doc.rust-lang.org/std/io/struct.Cursor.html#method.get_mut)
- [`std::ptr::Unique::get_mut`](https://doc.rust-lang.org/std/ptr/struct.Unique.html#method.get_mut)
- [`std::sync::PoisonError::get_mut`](https://doc.rust-lang.org/std/sync/struct.PoisonError.html#method.get_mut)
- [`std::sync::atomic::AtomicBool::get_mut`](https://doc.rust-lang.org/std/sync/atomic/struct.AtomicBool.html#method.get_mut)
- [`std::collections::hash_map::OccupiedEntry::get_mut`](https://doc.rust-lang.org/std/collections/hash_map/struct.OccupiedEntry.html#method.get_mut)
- [`<[_]>::get_unchecked`](https://doc.rust-lang.org/std/primitive.slice.html#method.get_unchecked)


<a id="c-feature"></a>
## Feature names are free of placeholder words (C-FEATURE)

Do not include words in the name of a [Cargo feature] that convey zero meaning,
as in `use-abc` or `with-abc`. Name the feature `abc` directly.

[Cargo feature]: http://doc.crates.io/manifest.html#the-features-section

This arises most commonly for crates that have an optional dependency on the
Rust standard library. The canonical way to do this correctly is:

```toml
# In Cargo.toml

[features]
default = ["std"]
std = []
```

```rust
// In lib.rs

#![cfg_attr(not(feature = "std"), no_std)]
```

Do not call the feature `use-std` or `with-std` or any creative name that is not
`std`. This naming convention aligns with the naming of implicit features
inferred by Cargo for optional dependencies. Consider crate `x` with optional
dependencies on Serde and on the Rust standard library:

```toml
[package]
name = "x"
version = "0.1.0"

[features]
std = ["serde/std"]

[dependencies]
serde = { version = "1.0", optional = true }
```

When we depend on `x`, we can enable the optional Serde dependency with
`features = ["serde"]`. Similarly we can enable the optional standard library
dependency with `features = ["std"]`. The implicit feature inferred by Cargo for
the optional dependency is called `serde`, not `use-serde` or `with-serde`, so
we like for explicit features to behave the same way.

As a related note, Cargo requires that features are additive so a feature named
negatively like `no-abc` is practically never correct.
