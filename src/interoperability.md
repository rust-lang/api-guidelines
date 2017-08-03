# Interoperability


<a id="c-common-traits"></a>
## Types eagerly implement common traits (C-COMMON-TRAITS)

Rust's trait system does not allow _orphans_: roughly, every `impl` must live
either in the crate that defines the trait or the implementing type.
Consequently, crates that define new types should eagerly implement all
applicable, common traits.

To see why, consider the following situation:

* Crate `std` defines trait `Display`.
* Crate `url` defines type `Url`, without implementing `Display`.
* Crate `webapp` imports from both `std` and `url`,

There is no way for `webapp` to add `Display` to `url`, since it defines
neither. (Note: the newtype pattern can provide an efficient, but inconvenient
workaround.

The most important common traits to implement from `std` are:

- [`Copy`](https://doc.rust-lang.org/std/marker/trait.Copy.html)
- [`Clone`](https://doc.rust-lang.org/std/clone/trait.Clone.html)
- [`Eq`](https://doc.rust-lang.org/std/cmp/trait.Eq.html)
- [`PartialEq`](https://doc.rust-lang.org/std/cmp/trait.PartialEq.html)
- [`Ord`](https://doc.rust-lang.org/std/cmp/trait.Ord.html)
- [`PartialOrd`](https://doc.rust-lang.org/std/cmp/trait.PartialOrd.html)
- [`Hash`](https://doc.rust-lang.org/std/hash/trait.Hash.html)
- [`Debug`](https://doc.rust-lang.org/std/fmt/trait.Debug.html)
- [`Display`](https://doc.rust-lang.org/std/fmt/trait.Display.html)
- [`Default`](https://doc.rust-lang.org/std/default/trait.Default.html)

Note that it is common and expected for types to implement both
`Default` and an empty `new` constructor. `new` is the constructor
convention in Rust, and users expect it to exist, so if it is
reasonable for the basic constructor to take no arguments, then it
should, even if it is functionally identical to `default`.


<a id="c-conv-traits"></a>
## Conversions use the standard traits `From`, `AsRef`, `AsMut` (C-CONV-TRAITS)

The following conversion traits should be implemented where it makes sense:

- [`From`](https://doc.rust-lang.org/std/convert/trait.From.html)
- [`TryFrom`](https://doc.rust-lang.org/std/convert/trait.TryFrom.html)
- [`AsRef`](https://doc.rust-lang.org/std/convert/trait.AsRef.html)
- [`AsMut`](https://doc.rust-lang.org/std/convert/trait.AsMut.html)

The following conversion traits should never be implemented:

- [`Into`](https://doc.rust-lang.org/std/convert/trait.Into.html)
- [`TryInto`](https://doc.rust-lang.org/std/convert/trait.TryInto.html)

These traits have a blanket impl based on `From` and `TryFrom`. Implement those
instead.

### Examples from the standard library

- `From<u16>` is implemented for `u32` because a smaller integer can always be
  converted to a bigger integer.
- `From<u32>` is *not* implemented for `u16` because the conversion may not be
  possible if the integer is too big.
- `TryFrom<u32>` is implemented for `u16` and returns an error if the integer is
  too big to fit in `u16`.
- [`From<Ipv6Addr>`] is implemented for [`IpAddr`], which is a type that can
  represent both v4 and v6 IP addresses.

[`From<Ipv6Addr>`]: https://doc.rust-lang.org/std/net/struct.Ipv6Addr.html
[`IpAddr`]: https://doc.rust-lang.org/std/net/enum.IpAddr.html


<a id="c-collect"></a>
## Collections implement `FromIterator` and `Extend` (C-COLLECT)

[`FromIterator`] and [`Extend`] enable collections to be used conveniently with
the following iterator methods:

[`FromIterator`]: https://doc.rust-lang.org/std/iter/trait.FromIterator.html
[`Extend`]: https://doc.rust-lang.org/std/iter/trait.Extend.html

- [`Iterator::collect`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.collect)
- [`Iterator::partition`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.partition)
- [`Iterator::unzip`](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.unzip)

`FromIterator` is for creating a new collection containing items from an
iterator, and `Extend` is for adding items from an iterator onto an existing
collection.

### Examples from the standard library

- [`Vec<T>`] implements both `FromIterator<T>` and `Extend<T>`.

[`Vec<T>`]: https://doc.rust-lang.org/std/vec/struct.Vec.html


<a id="c-serde"></a>
## Data structures implement Serde's `Serialize`, `Deserialize` (C-SERDE)

Types that play the role of a data structure should implement [`Serialize`] and
[`Deserialize`].

An example of a type that plays the role of a data structure is
[`linked_hash_map::LinkedHashMap`].

An example of a type that does not play the role of a data structure is
[`byteorder::LittleEndian`].

[`Serialize`]: https://docs.serde.rs/serde/trait.Serialize.html
[`Deserialize`]: https://docs.serde.rs/serde/trait.Deserialize.html
[`byteorder::LittleEndian`]: https://docs.rs/byteorder/1.0.0/byteorder/enum.LittleEndian.html
[`linked_hash_map::LinkedHashMap`]: https://docs.rs/linked-hash-map/0.4.2/linked_hash_map/struct.LinkedHashMap.html


<a id="c-serde-cfg"></a>
## Crate has a `"serde"` cfg option that enables Serde (C-SERDE-CFG)

If the crate relies on `serde_derive` to provide Serde impls, the name of the
cfg can still be simply `"serde"` by using [this workaround]. Do not use a
different name for the cfg like `"serde_impls"` or `"serde_serialization"`.

[this workaround]: https://github.com/serde-rs/serde/blob/v1.0.0/serde/src/lib.rs#L222-L260


<a id="c-send-sync"></a>
## Types are `Send` and `Sync` where possible (C-SEND-SYNC)

[`Send`] and [`Sync`] are automatically implemented when the compiler determines
it is appropriate.

[`Send`]: https://doc.rust-lang.org/std/marker/trait.Send.html
[`Sync`]: https://doc.rust-lang.org/std/marker/trait.Sync.html

In types that manipulate raw pointers, be vigilant that the `Send` and `Sync`
status of your type accurately reflects its thread safety characteristics. Tests
like the following can help catch unintentional regressions in whether the type
implements `Send` or `Sync`.

```rust
#[test]
fn test_send() {
    fn assert_send<T: Send>() {}
    assert_send::<MyStrangeType>();
}

#[test]
fn test_sync() {
    fn assert_sync<T: Sync>() {}
    assert_sync::<MyStrangeType>();
}
```


<a id="c-send-sync-err"></a>
## Error types are `Send` and `Sync` (C-SEND-SYNC-ERR)

An error that is not `Send` cannot be returned by a thread run with
[`thread::spawn`]. An error that is not `Sync` cannot be passed across threads
using an [`Arc`]. These are common requirements for basic error handling in a
multithreaded application.

[`thread::spawn`]: https://doc.rust-lang.org/std/thread/fn.spawn.html
[`Arc`]: https://doc.rust-lang.org/std/sync/struct.Arc.html

`Send` and `Sync` are also important for being able to package a custom error
into an IO error using [`std::io::Error::new`], which requires a trait bound of
`Error + Send + Sync`.

[`std::io::Error::new`]: https://doc.rust-lang.org/std/io/struct.Error.html#method.new

One place to be vigilant about this guideline is in functions that return Error
trait objects, for example [`reqwest::Error::get_ref`]. Typically `Error + Send
+ Sync + 'static` will be the most useful for callers. The addition of `'static`
allows the trait object to be used with [`Error::downcast_ref`].

[`reqwest::Error::get_ref`]: https://docs.rs/reqwest/0.7.2/reqwest/struct.Error.html#method.get_ref
[`Error::downcast_ref`]: https://doc.rust-lang.org/std/error/trait.Error.html#method.downcast_ref-2


<a id="c-meaningful-err"></a>
## Error types are meaningful, not `()` (C-MEANINGFUL-ERR)

When defining functions that return `Result`, and the error carries no
useful additional information, do not use `()` as the error type. `()`
does not implement `std::error::Error`, and this causes problems for
callers that expect to be able to convert errors to `Error`. Common
error handling libraries like [error-chain] expect errors to implement
`Error`.

[error-chain]: https://docs.rs/error-chain

Instead, define a meaningful error type specific to your crate.

### Examples from the standard library

- [`ParseBoolError`] is returned when failing to parse a bool from a string.

[`ParseBoolError`]: https://doc.rust-lang.org/std/str/struct.ParseBoolError.html


<a id="c-num-fmt"></a>
## Binary number types provide `Hex`, `Octal`, `Binary` formatting (C-NUM-FMT)

- [`std::fmt::UpperHex`](https://doc.rust-lang.org/std/fmt/trait.UpperHex.html)
- [`std::fmt::LowerHex`](https://doc.rust-lang.org/std/fmt/trait.LowerHex.html)
- [`std::fmt::Octal`](https://doc.rust-lang.org/std/fmt/trait.Octal.html)
- [`std::fmt::Binary`](https://doc.rust-lang.org/std/fmt/trait.Binary.html)

These traits control the representation of a type under the `{:X}`, `{:x}`,
`{:o}`, and `{:b}` format specifiers.

Implement these traits for any number type on which you would consider doing
bitwise manipulations like `|` or `&`. This is especially appropriate for
bitflag types. Numeric quantity types like `struct Nanoseconds(u64)` probably do
not need these.
