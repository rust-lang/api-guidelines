***NB: This is in real bad shape. Sorry***

# Rust API guidelines

* [Checklist](#checklist)
* [Naming](#naming)
* [Architecture](#architecture)
* [Containers](#containers)
* [Ownership and resource management](#ownership)
* [Error handling](#errors)
* [Documentation](#docs)
* [Unsorted guidelines](#unsorted)
* [External links](#external)


<a id="checklist"></a>
## Crate conformance checklist

- Naming
  - [ ] Follow general naming conventions per RFC 430 ([C-NAME])
  - [ ] Ad-hoc conversions should follow `as_`, `to_`, `into_` conventions ([C-CONV])
- Architecture
  - [ ] Common functionality should be reexported at the crate level ([C-REEXPORT])
  - [ ] Use the module hierarchy to organize APIs ([C-MODS])
  - [ ] Common functionality should be exported at the module level ([C-MOD-EXPORT])
  - [ ] Define types and impls together ([C-TOGETHER])
- Containers
  - [ ] Consider `FromIterator` and `Extend` for collections ([C-COLLECTIONS-TRAITS])
  - [ ] Implement standard conversion traits `From`, `TryFrom`, `Into`, `AsRef`, `AsMut` ([C-CONV-TRAITS])
- Ownership and resource management
- Error handling
- Documentation
  - [ ] Crate level docs are thorough and include exampls ([C-CRATE-DOC])
  - [ ] There are sufficient examples ([C-EXAMPLES])
  - [ ] Function docs include panic conditions in "Panics" section ([C-PANIC-DOC])
  - [ ] Function docs include error conditions in "Errors" section ([C-ERROR-DOC])
  - [ ] Cargo.toml publishes CI badges for tier 1 platforms ([C-CI])
  - [ ] Cargo.toml includes all common headers ([C-CARGO-HEADERS])
    - authors, description, documentation, homepage
    - repository, readme, keywords, categories, license
  - [ ] Crate contains html_root_url attribute ([C-HTML-ROOT])
- Unsorted guidelines
  - [ ] Eagerly implement common traits ([C-COMMON-TRAITS])
    - `Copy`, `Clone`, `Eq`, `PartialEq`, `Ord`, `PartialOrd`
    - `Hash` `Debug`, `Display`
  - [ ] All public types implement `Debug` ([C-DEBUG])
  - [ ] Most types should implement serde's `Serialize`, `Deserialize` ([C-SERDE])
  - [ ] Crate has a `serde` cfg option that enables serde ([C-SERDE-CFG])
  - [ ] Public dependencies must be stable ([C-PUB-DEP])
  - [ ] Crate and its dependencies have a permissive license ([C-PERMISSIVE])
  - [ ] Public types should impl `Default` if reasonable ([C-DEFAULT])
  - [ ] Cargo.toml should contain complete metadata ([C-TOML])
  - [ ] Single-element containers may implement `unwrap` ([C-UNWRAP])
  - [ ] Single-element should implement appropriate getters and setters ([C-GETTERS])
  - [ ] Structs should have private fields ([C-STRUCT-PRIVATE])
  - [ ] Smart pointers should not add inherent methods ([C-SMART-METHODS])
  - [ ] Associate conversions with the most specific type involved. ([C-CONV-SPECIFIC])
  - [ ] Methods that produce iterators should follow `iter`, `iter_mut`, `into_iter` ([C-ITER])
  - [ ] The name of an iterator type should be the same as the method that produces it ([C-ITER-NAME])
  - [ ] Use correct ownership suffixes, `_mut`, `_ref` ([C-OWN-SUFFIX])
  - [ ] Prefer methods to fuctions if there is a clear receiever ([C-PREFER-METHODS])
  - [ ] Return intermediate results to avoid duplicate work ([C-INTERMEDIATE])
  - [ ] Let the caller decide where to copy and place data ([C-CALLER-CONTROL])
  - [ ] Use generics to minimize assumptions about function parameters ([C-GENERIC-ARGS])
  - [ ] Prefer passing by reference ([C-BY-REF])
  - [ ] Don't use out parameters ([C-NO-OUT])
  - [ ] Validate arguments ([C-VALIDATE])
  - [ ] Know whether a trait will be used as an object. ([C-OBJ])
  - [ ] Implement `Send` and `Sync` when possible ([C-SEND-SYNC])
  - [ ] Error types should be `Send` and `Sync` ([C-SEND-SYNC-ERRORS])
  - [ ] Do not overload operators in surprising ways ([C-BAD-OVERLOAD])
  - [ ] Do not abuse `Deref` and `DerefMut` ([C-BAD-DEREF])
  - [ ] Do not fail within a `Deref`/`DerefMut` implementation. ([C-DEREF-FAIL])
  - [ ] Prefer trait-bounded generics to objects and virtual dispatch ([C-PREFER-GENERICS])
  - [ ] Prefer trait objects to generics ([C-PREFER-OBJECTS])
  - [ ] Use custom types, not `bool` and `Option` ([C-CUSTOM-TYPES])
  - [ ] Use `bitflags` for sets of flags, not enums ([C-BITFLAGS])
  - [ ] Use newtypes to provide static distinctions. ([C-NEWTYPE])
  - [ ] Use newtypes for encapsulation ([C-NEWTYPE-HIDE])
  - [ ] Use the builder pattern for complex value construction ([C-BUILDER])
  - [ ] Define constructors as static, inherent methods. ([C-CTOR])
  - [ ] Provide constructors for passive `struct`s with defaults. ([C-EMPTY-CTOR])
  - [ ] Destructors must not fail. ([C-DTOR-FAIL])
  - [ ] Destructors should not block. ([C-DTOR-BLOCK])


<a id="naming"></a>
## Naming

[C-NAME]: #c-name
<a id="c-name"></a>
### Follow general naming conventions per RFC 430 (C-NAME)

Basic Rust naming conventions are described in [RFC 430].

In general, Rust tends to use `CamelCase` for "type-level" constructs
(types and traits) and `snake_case` for "value-level" constructs. More
precisely:

| Item | Convention |
| ---- | ---------- |
| Crates | `snake_case` (but prefer single word) |
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
| Lifetimes | short, lowercase: `'a` |

In `CamelCase`, acronyms count as one word: use `Uuid` rather than
`UUID`.  In `snake_case`, acronyms are lower-cased: `is_xid_start`.

In `snake_case` or `SCREAMING_SNAKE_CASE`, a "word" should never
consist of a single letter unless it is the last "word". So, we have
`btree_map` rather than `b_tree_map`, but `PI_2` rather than `PI2`.

[C-CONV]: #c-conv
<a id="c-conv"></a>
### Ad-hoc conversions should follow `as_`, `to_`, `into_` conventions (C-CONV)

Conversions should be provided as methods, with names prefixed as follows:

| Prefix | Cost | Consumes convertee |
| ------ | ---- | ------------------ |
| `as_` | Free | No |
| `to_` | Expensive | No |
| `into_` | Variable | Yes |

For example:

- `as_bytes()` gives a `&[u8]` view into a `&str`, which is a no-op.
- `to_owned()` copies a `&str` to a new `String`.
- `into_bytes()` consumes a `String` and yields the underlying
  `Vec<u8>`, which is a no-op.

Conversions prefixed `as_` and `into_` typically _decrease abstraction_, either
exposing a view into the underlying representation (`as`) or deconstructing data
into its underlying representation (`into`). Conversions prefixed `to_`, on the
other hand, typically stay at the same level of abstraction but do some work to
change one representation into another.

More examples:

- [`Result::as_ref`](https://doc.rust-lang.org/std/result/enum.Result.html#method.as_ref)
- [`RefCell::as_ptr`](https://doc.rust-lang.org/std/cell/struct.RefCell.html#method.as_ptr)
- [`Path::to_str`](https://doc.rust-lang.org/std/path/struct.Path.html#method.to_str)
- [`slice::to_vec`](https://doc.rust-lang.org/std/primitive.slice.html#method.to_vec)
- [`Option::into_iter`](https://doc.rust-lang.org/std/option/enum.Option.html#method.into_iter)
- [`AtomicBool::into_inner`](https://doc.rust-lang.org/std/sync/atomic/struct.AtomicBool.html#method.into_inner)


<a id="architecture"></a>
## Architecture

[C-REEXPORT]: #c-reexport
<a id="c-reexport"></a>
### Common functionality should be reexported at the crate level (C-REEXPORT)

Crates `pub use` the most common types for convenience, so that
clients do not have to remember or write the crate's module hierarchy
to use these types.

[C-MODS]: #c-mods
<a id="c-mods"></a>
### Use the module hierarchy to organize APIs (C-MODS)

[C-MOD-EXPORT]: #c-mod-export
<a id="c-mod-export"></a>
### Common functionality should be exported at the module level (C-MOD-EXPORT)

For example,
[`IoError`](http://doc.rust-lang.org/std/io/struct.IoError.html)
is defined in `io/mod.rs`, since it pertains to the entirety of `io`,
while
[`TcpStream`](http://doc.rust-lang.org/std/io/net/tcp/struct.TcpStream.html)
is defined in `io/net/tcp.rs` and reexported in the `io` module.

[C-TOGETHER]: #c-together
<a id="c-together"></a>
### Define types and impls together (C-TOGETHER)

Type definitions and the functions/methods that operate on them should be
defined together in a single module, with the type appearing above the
functions/methods.

TODO: This doesn't impact the public API? Should we really consider it?


<a id="containers"></a>
## Containers

[C-COLLECTIONS-TRAITS]: #c-collections-traits
<a id="c-collections-traits"></a>
### Consider `FromIterator` and `Extend` for collections (C-COLLECTIONS-TRAITS)

[C-CONV-TRAITS]: #c-conv-traits
<a id="c-conv-traits"></a>
### Implement standard conversion traits `From`, `TryFrom`, `Into`, `AsRef`, `AsMut` (C-CONV-TRAITS)


<a id="ownership"></a>
## Ownership and resource management


<a id="errors"></a>
## Error handling


<a id="documentation"></a>
## Documentation

[C-DOC]: #c-doc
<a id="c-doc"></a>
### The crate should follow documentation convention RFCS (C-DOC)

As described in [RFC 505], [RFC 1687], [RFC 1574].

In particular:

- Crates should have full top-level documentation ([RFC 505]).
- Crates should use headings everywhere appropriate: 'Examples',
  'Panics', 'Errors', 'Safety', 'Aborts', 'Undefined Behavior'.
- Everything should have at least one example.
- Relevant things should be explicitly hyperlinked.

[C-CRATE-DOC]: #c-crate-doc
<a id="c-crate-doc"></a>
### Crate level docs are thorough and include examples (C-CRATE-DOC)

[C-EXAMPLES]: #c-examples
<a id="c-examples"></a>
### All items have a rustdoc example (C-EXAMPLES)

[C-PANIC-DOC]: #c-panic-doc
<a id="c-panic-doc"></a>
### Function docs include panic conditions in "Panics" section (C-PANIC-DOC)

[C-ERROR-DOC]: #c-error-doc
<a id="c-error-doc"></a>
### Function docs include error conditions in "Errors" section (C-ERROR-DOC)

[C-CI]: #c-ci
<a id="c-ci"></a>
### Cargo.toml publishes CI badges for tier 1 platforms (C-CI)

[C-CARGO-HEADERS]: #c-cargo-headers
<a id="c-cargo-headers"></a>
### Cargo.toml includes all common headers (C-CARGO-HEADERS)

[C-HTML-ROOT]: #c-html-root
<a id="c-html-root"></a>
### Crate contains html_root_url attribute (C-HTML-ROOT)


<a id="unsorted"></a>
## Unsorted guidelines

[C-COMMON-TRAITS]: #c-common-traits
<a id="c-common-traits"></a>
### Eagerly implement common traits (C-COMMON-TRAITS)

Rust's trait system does not allow _orphans_: roughly, every `impl` must live
either in the crate that defines the trait or the implementing
type. Consequently, crates that define new types should eagerly implement all
applicable, common traits.

To see why, consider the following situation:

* Crate `std` defines trait `Show`.
* Crate `url` defines type `Url`, without implementing `Show`.
* Crate `webapp` imports from both `std` and `url`,

There is no way for `webapp` to add `Show` to `url`, since it defines neither.
(Note: the newtype pattern can provide an efficient, but inconvenient
workaround; see [newtype for views](../types/newtype.md))

The most important common traits to implement from `std` are:

- `Copy`, `Clone`
- `Eq`, `PartialEq`, `Ord`, `PartialOrd`
- `Debug`, `Display`
- `Hash`

[C-DEBUG]: #c-debug
<a id="c-debug"></a>
### All public types implement `Debug` (C-DEBUG)

If there are exceptions, they are rare.

[C-SERDE]: #c-serde
<a id="c-serde"></a>
### Most types should implement serde's `Serialize`, `Deserialize` (C-SERDE)

[C-SERDE-CFG]: #c-serde-cfg
<a id="c-serde-cfg"></a>
### Crate has a serde cfg option that enables serde (C-SERDE-CFG)

[C-PUB-DEP]: #c-pub-dep
<a id="c-pub-dep"></a>
### Public dependencies must be stable (C-PUB-DEP)

[C-PERMISSIVE]: #c-permissive
<a id="c-permissive"></a>
### Crate and its dependencies have a permissive license (C-PERMISSIVE)

[C-DEFAULT]: #c-default
<a id="c-default"></a>
### Public types should impl `Default` if reasonable (C-DEFAULT)

[C-TOML]: #c-toml
<a id="c-toml"></a>
### Cargo.toml should contain complete metadata (C-TOML)

[C-UNWRAP]: #c-unwrap
<a id="c-unwrap"></a>
### Single-element containers may implement `unwrap` (C-UNWRAP)

[C-GETTERS]: #c-getters
<a id="c-getters"></a>
### Single-element should implement appropriate getters and setters (C-GETTERS)

Single-element contains where accessing the element cannot fail should
implement `get` and `get_mut`, with the signatures

```rust
fn get(&self) -> &V;
fn get_mut(&mut self) -> &mut V;
```

Single-element containers where the element is Copy (e.g. Cell-like containers)
should instead return the value directly, and not implement a mutable accessor:

```rust
fn get(&self) -> V;
```

Externally-mutable containers should have a mutable setter:

```rust
fn set(&mut self);
```

Internally-mutable containers should have a shared setter:

```rust
fn set(&self);
```

For getters that do runtime validation, consider adding unsafe
`_unchecked` variants:

```rust
unsafe fn get_unchecked(&self) -> &V;
```

[C-STRUCT-PRIVATE]: #c-struct-private
<a id="c-struct-private"></a>
### Structs should have private fields (C-STRUCT-PRIVATE)

Making a field public is a strong commitment: it pins down a representation
choice, _and_ prevents the type from providing any validation or maintaining any
invariants on the contents of the field, since clients can mutate it arbitrarily.

Public fields are most appropriate for `struct` types in the C spirit: compound,
passive data structures. Otherwise, consider providing getter/setter methods
and hiding fields instead.


[C-SMART-METHODS]: #c-smart-methods
<a id="c-smart-methods"></a>
### Smart pointers should not add inherent methods (C-SMART-METHODS)

[C-CONV-SPECIFIC]: #c-conv-specific
<a id="c-conv-specific"></a>
### Associate conversions with the most specific type involved. (C-CONV-SPECIFIC)

When in doubt, prefer `to_`/`as_`/`into_` to `from_`, because they are
more ergonomic to use (and can be chained with other methods).

For many conversions between two types, one of the types is clearly more
"specific": it provides some additional invariant or interpretation that is not
present in the other type. For example, `str` is more specific than `&[u8]`,
since it is a utf-8 encoded sequence of bytes.

Conversions should live with the more specific of the involved types. Thus,
`str` provides both the `as_bytes` method and the `from_utf8` constructor for
converting to and from `&[u8]` values. Besides being intuitive, this convention
avoids polluting concrete types like `&[u8]` with endless conversion methods.

[C-ITER]: #c-iter
<a id="c-iter"></a>
### Methods that produce iterators should follow `iter`, `iter_mut`, `into_iter` (C-ITER)

Per [RFC 199].

For a container with elements of type `U`, iterator methods should be named:

```rust
fn iter(&self) -> T           // where T implements Iterator<&U>
fn iter_mut(&mut self) -> T   // where T implements Iterator<&mut U>
fn into_iter(self) -> T       // where T implements Iterator<U>
```

The default iterator variant yields shared references `&U`.

[C-ITER-NAME]: #c-iter-name
<a id="c-iter-name"></a>
### The name of an iterator type should be the same as the method that produces it (C-ITER-NAME)

For example:

* `iter` should yield an `Iter`
* `iter_mut` should yield an `IterMut`
* `into_iter` should yield an `IntoIter`
* `keys` should yield `Keys`

These type names make the most sense when prefixed with their owning module,
e.g. `vec::IntoIter`.

[C-OWN-SUFFIX]: #c-own-suffix
<a id="c-own-suffix"></a>
### Use correct ownership suffixes, `_mut`, `_ref` (C-OWN-SUFFIX)

Functions often come in multiple variants: immutably borrowed, mutably
borrowed, and owned.

The right default depends on the function in question. Variants should
be marked through suffixes.

#### Exceptions

In the case of iterators, the moving variant can also be understood as
an `into` conversion, `into_iter`, and `for x in v.into_iter()` reads
arguably better than `for x in v.iter_move()`, so the convention is
`into_iter`.

For mutably borrowed variants, if the `mut` qualifier is part of a
type name (e.g. `as_mut_slice`), it should appear as it would appear
in the type.

#### Immutably borrowed by default

If `foo` uses/produces an immutable borrow by default, use:

* The `_mut` suffix (e.g. `foo_mut`) for the mutably borrowed variant.
* The `_move` suffix (e.g. `foo_move`) for the owned variant.

#### Owned by default

If `foo` uses/produces owned data by default, use:

* The `_ref` suffix (e.g. `foo_ref`) for the immutably borrowed variant.
* The `_mut` suffix (e.g. `foo_mut`) for the mutably borrowed variant.

[C-PREFER-METHODS]: #c-prefer-methods
<a id="c-prefer-methods"></a>
### Prefer methods to fuctions if there is a clear receiever (C-PREFER-METHODS)

Prefer

```rust
impl Foo {
    pub fn frob(&self, w: widget) { ... }
}
```

over

```rust
pub fn frob(foo: &Foo, w: widget) { ... }
```

for any operation that is clearly associated with a particular
type.

Methods have numerous advantages over functions:

* They do not need to be imported or qualified to be used: all you
  need is a value of the appropriate type.
* Their invocation performs autoborrowing (including mutable borrows).
* They make it easy to answer the question "what can I do with a value
  of type `T`" (especially when using rustdoc).
* They provide `self` notation, which is more concise and often more
  clearly conveys ownership distinctions.

[C-INTERMEDIATE]: #c-intermediate
<a id="c-intermediate"></a>
### Return intermediate results to avoid duplicate work (C-INTERMEDIATE)

Many functions that answer a question also compute interesting related data.  If
this data is potentially of interest to the client, consider exposing it in the
API.

Prefer

```rust
struct SearchResult {
    found: bool,          // item in container?
    expected_index: uint  // what would the item's index be?
}

fn binary_search(&self, k: Key) -> SearchResult
```
or

```rust
fn binary_search(&self, k: Key) -> (bool, uint)
```

over

```rust
fn binary_search(&self, k: Key) -> bool
```

#### Yield back ownership:

Prefer

```rust
fn from_utf8_owned(vv: Vec<u8>) -> Result<String, Vec<u8>>
```

over

```rust
fn from_utf8_owned(vv: Vec<u8>) -> Option<String>
```

The `from_utf8_owned` function gains ownership of a vector.  In the successful
case, the function consumes its input, returning an owned string without
allocating or copying. In the unsuccessful case, however, the function returns
back ownership of the original slice.

[C-CALLER-CONTROL]: #c-caller-control
<a id="c-caller-control"></a>
### Let the caller decide where to copy and place data (C-CALLER-CONTROL)

Prefer

```rust
fn foo(b: Bar) {
   // use b as owned, directly
}
```

over

```rust
fn foo(b: &Bar) {
    let b = b.clone();
    // use b as owned after cloning
}
```

If a function requires ownership of a value of unknown type `T`, but does not
otherwise need to make copies, the function should take ownership of the
argument (pass by value `T`) rather than using `.clone()`. That way, the caller
can decide whether to relinquish ownership or to `clone`.

Similarly, the `Copy` trait bound should only be demanded it when absolutely
needed, not as a way of signaling that copies should be cheap to make.

Prefer

```rust
fn foo(b: Bar) -> Bar { ... }
```

over

```rust
fn foo(b: Box<Bar>) -> Box<Bar> { ... }
```

[C-GENERIC-ARGS]: #c-generic-args
<a id="c-generic-args"></a>
### Use generics to minimize assumptions about function parameters (C-GENERIC-ARGS)

The fewer assumptions a function makes about its inputs, the more widely usable
it becomes.

Prefer

```rust
fn foo<T: Iterator<int>>(c: T) { ... }
```

over any of

```rust
fn foo(c: &[int]) { ... }
fn foo(c: &Vec<int>) { ... }
fn foo(c: &SomeOtherCollection<int>) { ... }
```

if the function only needs to iterate over the data.

More generally, consider using generics to pinpoint the assumptions a function
needs to make about its arguments.

On the other hand, generics can make it more difficult to read and understand a
function's signature. Aim for "natural" parameter types that a neither overly
concrete nor overly abstract. See the discussion on
[traits](../../traits/README.md) for more guidance.

[C-BY-REF]: #c-by-ref
<a id="c-by-ref"></a>
### Prefer passing by reference (C-BY-REF)

Prefer either of

```rust
fn foo(b: &Bar) { ... }
fn foo(b: &mut Bar) { ... }
```

over

```rust
fn foo(b: Bar) { ... }
```

That is, prefer borrowing arguments rather than transferring ownership, unless
ownership is actually needed.

[C-NO-OUT]: #c-no-out
<a id="c-no-out"></a>
### Don't use out parameters (C-NO-OUT)

Prefer

```rust
fn foo() -> (Bar, Bar)
```

over

```rust
fn foo(output: &mut Bar) -> Bar
```

for returning multiple `Bar` values.

Compound return types like tuples and structs are efficiently compiled
and do not require heap allocation. If a function needs to return
multiple values, it should do so via one of these types.

The primary exception: sometimes a function is meant to modify data
that the caller already owns, for example to re-use a buffer:

```rust
fn read(&mut self, buf: &mut [u8]) -> IoResult<uint>
```

[C-VALIDATE]: #c-validate
<a id="c-validate"></a>
### Validate arguments (C-VALIDATE)

Rust APIs do _not_ generally follow the
[robustness principle](http://en.wikipedia.org/wiki/Robustness_principle): "be
conservative in what you send; be liberal in what you accept".

Instead, Rust code should _enforce_ the validity of input whenever practical.

Enforcement can be achieved through the following mechanisms (listed
in order of preference).

#### Static enforcement:

Choose an argument type that rules out bad inputs.

For example, prefer

```rust
fn foo(a: ascii::Ascii) { ... }
```

over

```rust
fn foo(a: u8) { ... }
```

Note that
[`ascii::Ascii`](http://static.rust-lang.org/doc/master/std/ascii/struct.Ascii.html)
is a _wrapper_ around `u8` that guarantees the highest bit is zero; see
[newtype patterns](../types/newtype.md) for more details on creating typesafe wrappers.

Static enforcement usually comes at little run-time cost: it pushes the
costs to the boundaries (e.g. when a `u8` is first converted into an
`Ascii`). It also catches bugs early, during compilation, rather than through
run-time failures.

On the other hand, some properties are difficult or impossible to
express using types.

#### Dynamic enforcement:

Validate the input as it is processed (or ahead of time, if necessary).  Dynamic
checking is often easier to implement than static checking, but has several
downsides:

1. Runtime overhead (unless checking can be done as part of processing the input).
2. Delayed detection of bugs.
3. Introduces failure cases, either via `fail!` or `Result`/`Option` types (see
   the [error handling guidelines](../../errors/README.md)), which must then be
   dealt with by client code.

#### Dynamic enforcement with `debug_assert!`:

Same as dynamic enforcement, but with the possibility of easily turning off
expensive checks for production builds.

#### Dynamic enforcement with opt-out:

Same as dynamic enforcement, but adds sibling functions that opt out of the
checking.

The convention is to mark these opt-out functions with a suffix like
`_unchecked` or by placing them in a `raw` submodule.

The unchecked functions can be used judiciously in cases where (1) performance
dictates avoiding checks and (2) the client is otherwise confident that the
inputs are valid.

[C-OBJ]: #c-obj
<a id="c-obj"></a>
### Know whether a trait will be used as an object. (C-OBJ)

Trait objects have some [significant limitations](objects.md): methods
invoked through a trait object cannot use generics, and cannot use
`Self` except in receiver position.

When designing a trait, decide early on whether the trait will be used
as an [object](objects.md) or as a [bound on generics](generics.md);
the tradeoffs are discussed in each of the linked sections.

If a trait is meant to be used as an object, its methods should take
and return trait objects rather than use generics.

[C-SEND-SYNC]: #c-send-sync
<a id="c-send-sync"></a>
### Implement `Send` and `Sync` when possible (C-SEND-SYNC)

[C-SEND-SYNC-ERRORS]: #c-send-sync-errors
<a id="c-send-sync-errors"></a>
### Error types should be `Send` and `Sync` (C-SEND-SYNC-ERRORS)

[C-BAD-OVERLOAD]: #c-bad-overload
<a id="c-bad-overload"></a>
### Do not overload operators in surprising ways (C-BAD-OVERLOAD)

Operators with built in syntax (`*`, `|`, and so on) can be provided for a type
by implementing the traits in `core::ops`. These operators come with strong
expectations: implement `Mul` only for an operation that bears some resemblance
to multiplication (and shares the expected properties, e.g. associativity), and
so on for the other traits.

[C-BAD-DEREF]: #c-bad-deref
<a id="c-bad-deref"></a>
### Do not abuse `Deref` and `DerefMut` (C-BAD-DEREF)

The `Deref` traits are used implicitly by the compiler in many circumstances,
and interact with method resolution. The relevant rules are designed
specifically to accommodate smart pointers, and so the traits should be used
only for that purpose.

[C-DEREF-FAIL]: #c-deref-fail
<a id="c-deref-fail"></a>
### Do not fail within a `Deref`/`DerefMut` implementation. (C-DEREF-FAIL)

Because the `Deref` traits are invoked implicitly by the compiler in sometimes
subtle ways, failure during dereferencing can be extremely confusing. If a
dereference might not succeed, target the `Deref` trait as a `Result` or
`Option` type instead.


[C-PREFER-GENERICS]: #c-prefer-generics
<a id="c-prefer-generics"></a>
### Prefer trait-bounded generics to objects and virtual dispatch (C-PREFER-GENERICS)

The most widespread use of traits is for writing generic functions or types. For
example, the following signature describes a function for consuming any iterator
yielding items of type `A` to produce a collection of `A`:

```rust
fn from_iter<T: Iterator<A>>(iterator: T) -> SomeCollection<A>
```

Here, the `Iterator` trait is specifies an interface that a type `T` must
explicitly implement to be used by this generic function.

**Pros**:

* _Reusability_. Generic functions can be applied to an open-ended collection of
  types, while giving a clear contract for the functionality those types must
  provide.
* _Static dispatch and optimization_. Each use of a generic function is
  specialized ("monomorphized") to the particular types implementing the trait
  bounds, which means that (1) invocations of trait methods are static, direct
  calls to the implementation and (2) the compiler can inline and otherwise
  optimize these calls.
* _Inline layout_. If a `struct` and `enum` type is generic over some type
  parameter `T`, values of type `T` will be laid out _inline_ in the
  `struct`/`enum`, without any indirection.
* _Inference_. Since the type parameters to generic functions can usually be
  inferred, generic functions can help cut down on verbosity in code where
  explicit conversions or other method calls would usually be necessary. See the
  [overloading/implicits use case](#use-case:-limited-overloading-and/or-implicit-conversions)
  below.
* _Precise types_. Because generic give a _name_ to the specific type
  implementing a trait, it is possible to be precise about places where that
  exact type is required or produced. For example, a function

  ```rust
  fn binary<T: Trait>(x: T, y: T) -> T
  ```

  is guaranteed to consume and produce elements of exactly the same type `T`; it
  cannot be invoked with parameters of different types that both implement
  `Trait`.

**Cons**:

* _Code size_. Specializing generic functions means that the function body is
  duplicated. The increase in code size must be weighed against the performance
  benefits of static dispatch.
* _Homogeneous types_. This is the other side of the "precise types" coin: if
  `T` is a type parameter, it stands for a _single_ actual type. So for example
  a `Vec<T>` contains elements of a single concrete type (and, indeed, the
  vector representation is specialized to lay these out in line). Sometimes
  heterogeneous collections are useful; see
  [trait objects](#use-case:-trait-objects) below.
* _Signature verbosity_. Heavy use of generics can bloat function signatures.
  **[Ed. note]** This problem may be mitigated by some language improvements; stay tuned.

[C-PREFER-OBJECTS]: #c-prefer-objects
<a id="c-prefer-objects"></a>
### Prefer trait objects to generics (C-PREFER-OBJECTS)

Trait objects are useful primarily when _heterogeneous_ collections of objects
need to be treated uniformly; it is the closest that Rust comes to
object-oriented programming.

```rust
struct Frame  { ... }
struct Button { ... }
struct Label  { ... }

trait Widget  { ... }

impl Widget for Frame  { ... }
impl Widget for Button { ... }
impl Widget for Label  { ... }

impl Frame {
    fn new(contents: &[Box<Widget>]) -> Frame {
        ...
    }
}

fn make_gui() -> Box<Widget> {
    let b: Box<Widget> = box Button::new(...);
    let l: Box<Widget> = box Label::new(...);

    box Frame::new([b, l]) as Box<Widget>
}
```

By using trait objects, we can set up a GUI framework with a `Frame` widget that
contains a heterogeneous collection of children widgets.

**Pros**:

* _Heterogeneity_. When you need it, you really need it.
* _Code size_. Unlike generics, trait objects do not generate specialized
  (monomorphized) versions of code, which can greatly reduce code size.

**Cons**:

* _No generic methods_. Trait objects cannot currently provide generic methods.
* _Dynamic dispatch and fat pointers_. Trait objects inherently involve
  indirection and vtable dispatch, which can carry a performance penalty.
* _No Self_. Except for the method receiver argument, methods on trait objects
  cannot use the `Self` type.

[C-CUSTOM-TYPES]: #c-custom-types
<a id="c-custom-types"></a>
### Use custom types, not `bool` and `Option` (C-CUSTOM-TYPES)

Prefer

```rust
let w = Widget::new(Small, Round)
```

over

```rust
let w = Widget::new(true, false)
```

Core types like `bool`, `u8` and `Option` have many possible interpretations.

Use custom types (whether `enum`s, `struct`, or tuples) to convey
interpretation and invariants. In the above example,
it is not immediately clear what `true` and `false` are conveying without
looking up the argument names, but `Small` and `Round` are more suggestive.

Using custom types makes it easier to expand the
options later on, for example by adding an `ExtraLarge` variant.

See [the newtype pattern](newtype.md) for a no-cost way to wrap
existing types with a distinguished name.


[C-BITFLAGS]: #c-bitflags
<a id="c-bitflags"></a>
### Use `bitflags` for sets of flags, not enums (C-BITFLAGS)

Rust supports `enum` types with "custom discriminants":

~~~~
enum Color {
  Red = 0xff0000,
  Green = 0x00ff00,
  Blue = 0x0000ff
}
~~~~

Custom discriminants are useful when an `enum` type needs to be serialized to an
integer value compatibly with some other system/language. They support
"typesafe" APIs: by taking a `Color`, rather than an integer, a function is
guaranteed to get well-formed inputs, even if it later views those inputs as
integers.

An `enum` allows an API to request exactly one choice from among many. Sometimes
an API's input is instead the presence or absence of a set of flags. In C code,
this is often done by having each flag correspond to a particular bit, allowing
a single integer to represent, say, 32 or 64 flags. Rust's `std::bitflags`
module provides a typesafe way for doing so.

[C-NEWTYPE]: #c-newtype
<a id="c-newtype"></a>
### Use newtypes to provide static distinctions. (C-NEWTYPE)

Newtypes can statically distinguish between different interpretations of an
underlying type.

For example, a `f64` value might be used to represent a quantity in miles or in
kilometers. Using newtypes, we can keep track of the intended interpretation:

```rust
struct Miles(pub f64);
struct Kilometers(pub f64);

impl Miles {
    fn as_kilometers(&self) -> Kilometers { ... }
}
impl Kilometers {
    fn as_miles(&self) -> Miles { ... }
}
```

Once we have separated these two types, we can statically ensure that we do not
confuse them. For example, the function

```rust
fn are_we_there_yet(distance_travelled: Miles) -> bool { ... }
```

cannot accidentally be called with a `Kilometers` value. The compiler will
remind us to perform the conversion, thus averting certain
[catastrophic bugs](http://en.wikipedia.org/wiki/Mars_Climate_Orbiter).

[C-NEWTYPE-HIDE]: #c-newtype-hide
<a id="c-newtype-hide"></a>
### Use newtypes for encapsulation (C-NEWTYPE-HIDE)

A newtype can be used to hide representation details while making precise
promises to the client.

For example, consider a function `my_transform` that returns a compound iterator
type `Enumerate<Skip<vec::MoveItems<T>>>`. We wish to hide this type from the
client, so that the client's view of the return type is roughly `Iterator<(uint,
T)>`. We can do so using the newtype pattern:

```rust
struct MyTransformResult<T>(Enumerate<Skip<vec::MoveItems<T>>>);
impl<T> Iterator<(uint, T)> for MyTransformResult<T> { ... }

fn my_transform<T, Iter: Iterator<T>>(iter: Iter) -> MyTransformResult<T> {
    ...
}
```

Aside from simplifying the signature, this use of newtypes allows us to make a
expose and promise less to the client. The client does not know _how_ the result
iterator is constructed or represented, which means the representation can
change in the future without breaking client code.

> **[FIXME]** Interaction with auto-deref.

[C-BUILDER]: #c-builder
<a id="c-builder"></a>
### Use the builder pattern for complex value construction (C-BUILDER)

Some data structures are complicated to construct, due to their construction needing:

* a large number of inputs
* compound data (e.g. slices)
* optional configuration data
* choice between several flavors

which can easily lead to a large number of distinct constructors with
many arguments each.

If `T` is such a data structure, consider introducing a `T` _builder_:

1. Introduce a separate data type `TBuilder` for incrementally configuring a `T`
   value. When possible, choose a better name: e.g. `Command` is the builder for
   `Process`.
2. The builder constructor should take as parameters only the data _required_ to
   make a `T`.
3. The builder should offer a suite of convenient methods for configuration,
   including setting up compound inputs (like slices) incrementally.
   These methods should return `self` to allow chaining.
4. The builder should provide one or more "_terminal_" methods for actually building a `T`.

The builder pattern is especially appropriate when building a `T` involves side
effects, such as spawning a task or launching a process.

In Rust, there are two variants of the builder pattern, differing in the
treatment of ownership, as described below.

#### Non-consuming builders (preferred):

In some cases, constructing the final `T` does not require the builder itself to
be consumed. The follow variant on
[`std::io::process::Command`](http://static.rust-lang.org/doc/master/std/io/process/struct.Command.html)
is one example:

```rust
// NOTE: the actual Command API does not use owned Strings;
// this is a simplified version.

pub struct Command {
    program: String,
    args: Vec<String>,
    cwd: Option<String>,
    // etc
}

impl Command {
    pub fn new(program: String) -> Command {
        Command {
            program: program,
            args: Vec::new(),
            cwd: None,
        }
    }

    /// Add an argument to pass to the program.
    pub fn arg<'a>(&'a mut self, arg: String) -> &'a mut Command {
        self.args.push(arg);
        self
    }

    /// Add multiple arguments to pass to the program.
    pub fn args<'a>(&'a mut self, args: &[String])
                    -> &'a mut Command {
        self.args.push_all(args);
        self
    }

    /// Set the working directory for the child process.
    pub fn cwd<'a>(&'a mut self, dir: String) -> &'a mut Command {
        self.cwd = Some(dir);
        self
    }

    /// Executes the command as a child process, which is returned.
    pub fn spawn(&self) -> IoResult<Process> {
        ...
    }
}
```

Note that the `spawn` method, which actually uses the builder configuration to
spawn a process, takes the builder by immutable reference. This is possible
because spawning the process does not require ownership of the configuration
data.

Because the terminal `spawn` method only needs a reference, the configuration
methods take and return a mutable borrow of `self`.

##### The benefit

By using borrows throughout, `Command` can be used conveniently for both
one-liner and more complex constructions:

```rust
// One-liners
Command::new("/bin/cat").arg("file.txt").spawn();

// Complex configuration
let mut cmd = Command::new("/bin/ls");
cmd.arg(".");

if size_sorted {
    cmd.arg("-S");
}

cmd.spawn();
```

#### Consuming builders:

Sometimes builders must transfer ownership when constructing the final type
`T`, meaning that the terminal methods must take `self` rather than `&self`:

```rust
// A simplified excerpt from std::task::TaskBuilder

impl TaskBuilder {
    /// Name the task-to-be. Currently the name is used for identification
    /// only in failure messages.
    pub fn named(mut self, name: String) -> TaskBuilder {
        self.name = Some(name);
        self
    }

    /// Redirect task-local stdout.
    pub fn stdout(mut self, stdout: Box<Writer + Send>) -> TaskBuilder {
        self.stdout = Some(stdout);
        //   ^~~~~~ this is owned and cannot be cloned/re-used
        self
    }

    /// Creates and executes a new child task.
    pub fn spawn(self, f: proc():Send) {
        // consume self
        ...
    }
}
```

Here, the `stdout` configuration involves passing ownership of a `Writer`,
which must be transferred to the task upon construction (in `spawn`).

When the terminal methods of the builder require ownership, there is a basic tradeoff:

* If the other builder methods take/return a mutable borrow, the complex
  configuration case will work well, but one-liner configuration becomes
  _impossible_.

* If the other builder methods take/return an owned `self`, one-liners
  continue to work well but complex configuration is less convenient.

Under the rubric of making easy things easy and hard things possible, _all_
builder methods for a consuming builder should take and returned an owned
`self`. Then client code works as follows:

```rust
// One-liners
TaskBuilder::new().named("my_task").spawn(proc() { ... });

// Complex configuration
let mut task = TaskBuilder::new();
task = task.named("my_task_2"); // must re-assign to retain ownership

if reroute {
    task = task.stdout(mywriter);
}

task.spawn(proc() { ... });
```

One-liners work as before, because ownership is threaded through each of the
builder methods until being consumed by `spawn`. Complex configuration,
however, is more verbose: it requires re-assigning the builder at each step.

[C-CTOR]: #c-ctor
<a id="c-ctor"></a>
### Define constructors as static, inherent methods. (C-CTOR)

In Rust, "constructors" are just a convention:

```rust
impl<T> Vec<T> {
    pub fn new() -> Vec<T> { ... }
}
```

Constructors are static (no `self`) inherent methods for the type that they
construct. Combined with the practice of
[fully importing type names](../style/imports.md), this convention leads to
informative but concise construction:

```rust
use vec::Vec;

// construct a new vector
let mut v = Vec::new();
```

This convention also applied to conversion constructors (prefix `from` rather
than `new`).

[C-EMPTY-CTOR]: #c-empty-ctor
<a id="c-empty-ctor"></a>
### Provide constructors for passive `struct`s with defaults. (C-EMPTY-CTOR)

Given the `struct`

```rust
pub struct Config {
    pub color: Color,
    pub size:  Size,
    pub shape: Shape,
}
```

provide a constructor if there are sensible defaults:

```rust
impl Config {
    pub fn new() -> Config {
        Config {
            color: Brown,
            size: Medium,
            shape: Square,
        }
    }
}
```

which then allows clients to concisely override using `struct` update syntax:

```rust
Config { color: Red, .. Config::new() };
```

[C-DTOR-FAIL]: #c-dtor-fail
<a id="c-dtor-fail"></a>
### Destructors must not fail. (C-DTOR-FAIL)

Destructors are executed on task failure, and in that context a failing
destructor causes the program to abort.

Instead of failing in a destructor, provide a separate method for checking for
clean teardown, e.g. a `close` method, that returns a `Result` to signal
problems.

[C-DTOR-BLOCK]: #c-dtor-block
<a id="c-dtor-block"></a>
### Destructors should not block. (C-DTOR-BLOCK)

Similarly, destructors should not invoke blocking operations, which can make
debugging much more difficult. Again, consider providing a separate method for
preparing for an infallible, nonblocking teardown.


<a id="links"></a>
## External Links

- [RFC 199] - Ownership naming conventions
- [RFC 344] - Naming conventions
- [RFC 430] - Naming conventions
- [RFC 505] - Doc conventions
- [RFC 1574] - Doc conventions
- [RFC 1687] - Crate-level documentation

[RFC 344]: https://github.com/rust-lang/rfcs/blob/master/text/0344-conventions-galore.md
[RFC 430]: https://github.com/rust-lang/rfcs/blob/master/text/0430-finalizing-naming-conventions.md
[RFC 1687]: https://github.com/rust-lang/rfcs/pull/1687
[RFC 505]: https://github.com/rust-lang/rfcs/blob/master/text/0505-api-comment-conventions.md
[RFC 1574]: https://github.com/rust-lang/rfcs/blob/master/text/1574-more-api-documentation-conventions.md
[RFC 199]: https://github.com/rust-lang/rfcs/blob/master/text/0199-ownership-variants.md



# More criteria and questions

- Error types should implement Error
- 'inner'
- "don't overpromise"
- "let clients choose what to throw away"
- lossy conversions
- Rust-purity
- Tier-X platform support
- Test coverage
- CI standards
- Version number
- Serde implementation
  - byteorder example of crate that does not use serde
- README.md
- Licensing
- Examples
- Ecosystem interop
- no_std
- public dependencies must be 1.0
- public dependencies need to be 
- rules for picking unsafe pointer types (re Rc::into_raw)
- when to create Default impls? not for uninhabitable types
- how are compile time options documented?
- *const vs *mut and interior mutability ala Rc::into_raw
  - Shared type returns wrong type
- one-syscall rule
  - see create_dir vs create_dir_all https://github.com/rust-lang/rust/pull/39799
- why do we implement Default on byteorder::LittleEndian?
- phantom types have common impls (Copy, etc) re byteorder
  - even for phantom types, because of derive generates trait constraints
  - adding more common traits will be a breaking change because the ByteOrder trait won't implement them
  - serde_json uses a private supertrait to encapsulate this
