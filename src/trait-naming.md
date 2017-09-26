# Trait Naming Examples

## Adjectives

- `std::marker::Sized`

Marker for types with a constant size known at compile time.

- `std::marker::Sync` (internally synchronized)

Marker for types for which it is safe to share references between threads.

- `std::panic::UnwindSafe`

Marker for "panic safe" types.

## Verbs

- `std::marker::Send`

Marker for types that can be transferred across thread boundaries.

- `std::marker::Copy`

Marker for types whose values can be duplicated simply by copying bits.

- `std::borrow::Borrow`

Abstraction over all ways of borrowing data from a given type.

- `std::clone::Clone`

Abstraction over the ability to explicitly duplicate an object.

- `std::io::Write`

A trait for objects which are byte-oriented sinks.

- `serde::ser::Serialize`

A trait for objects that can be serialized into any data format.

## Nouns

- `std::iter::Iterator` 

An interface for dealing with iterators. Iterators are needed for performing an operation on the elements of a collection of some kind.

- `futures::future::Future`

An interface for dealing with futures. Futures are placeholders of a value that may become available at some later point in time.

- `futures::future::Executor`

An interface for spawning fresh futures. Implemented for "executors", or those types which can execute futures to completion.

- `std::hash::Hasher`

An interface for hashing an arbitrary stream of bytes. Instances of Hasher usually represent state that is changed while hashing data.

- `std::ops::Fn`

A version of the call operator that takes an immutable receiver.

- `std::fmt::Binary` & `std::fmt::Debug`

Both are format traits, which `format!` arguments ascribe to. These are format trait for the `b` and `?` character.

- `std::default::Default`

A trait for giving a type a useful default value.

## Prepositions or prepositional phrases.

- `std::convert::Into`

A conversion that consumes `self`, which may or may not be expensive.

- `std::convert::TryInto`

An attempted conversion that consumes `self`, which may or may not be expensive.

- `std::convert::AsRef`

A cheap reference-to-reference conversion. 

- `std::borrow::ToOwned`

A generalization of `Clone` to borrowed data.

- `std::convert::From`

Simple and safe type conversions in to `Self`.
