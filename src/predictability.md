# Predictability


<a id="c-smart-ptr"></a>
## Smart pointers do not add inherent methods (C-SMART-PTR)

For example, this is why the [`Box::into_raw`] function is defined the way it
is.

[`Box::into_raw`]: https://doc.rust-lang.org/std/boxed/struct.Box.html#method.into_raw

```rust
impl<T> Box<T> where T: ?Sized {
    fn into_raw(b: Box<T>) -> *mut T { /* ... */ }
}

let boxed_str: Box<str> = /* ... */;
let ptr = Box::into_raw(boxed_str);
```

If this were defined as an inherent method instead, it would be confusing at the
call site whether the method being called is a method on `Box<T>` or a method on
`T`.

```rust
impl<T> Box<T> where T: ?Sized {
    // Do not do this.
    fn into_raw(self) -> *mut T { /* ... */ }
}

let boxed_str: Box<str> = /* ... */;

// This is a method on str accessed through the smart pointer Deref impl.
boxed_str.chars()

// This is a method on Box<str>...?
boxed_str.into_raw()
```


<a id="c-conv-specific"></a>
## Conversions live on the most specific type involved (C-CONV-SPECIFIC)

When in doubt, prefer `to_`/`as_`/`into_` to `from_`, because they are more
ergonomic to use (and can be chained with other methods).

For many conversions between two types, one of the types is clearly more
"specific": it provides some additional invariant or interpretation that is not
present in the other type. For example, [`str`] is more specific than `&[u8]`,
since it is a UTF-8 encoded sequence of bytes.

[`str`]: https://doc.rust-lang.org/std/primitive.str.html

Conversions should live with the more specific of the involved types. Thus,
`str` provides both the [`as_bytes`] method and the [`from_utf8`] constructor
for converting to and from `&[u8]` values. Besides being intuitive, this
convention avoids polluting concrete types like `&[u8]` with endless conversion
methods.

[`as_bytes`]: https://doc.rust-lang.org/std/primitive.str.html#method.as_bytes
[`from_utf8`]: https://doc.rust-lang.org/std/str/fn.from_utf8.html


<a id="c-method"></a>
## Functions with a clear receiver are methods (C-METHOD)

Prefer

```rust
impl Foo {
    pub fn frob(&self, w: widget) { /* ... */ }
}
```

over

```rust
pub fn frob(foo: &Foo, w: widget) { /* ... */ }
```

for any operation that is clearly associated with a particular type.

Methods have numerous advantages over functions:

* They do not need to be imported or qualified to be used: all you need is a
  value of the appropriate type.
* Their invocation performs autoborrowing (including mutable borrows).
* They make it easy to answer the question "what can I do with a value of type
  `T`" (especially when using rustdoc).
* They provide `self` notation, which is more concise and often more clearly
  conveys ownership distinctions.


<a id="c-no-out"></a>
## Functions do not take out-parameters (C-NO-OUT)

Prefer

```rust
fn foo() -> (Bar, Bar)
```

over

```rust
fn foo(output: &mut Bar) -> Bar
```

for returning multiple `Bar` values.

Compound return types like tuples and structs are efficiently compiled and do
not require heap allocation. If a function needs to return multiple values, it
should do so via one of these types.

The primary exception: sometimes a function is meant to modify data that the
caller already owns, for example to re-use a buffer:

```rust
fn read(&mut self, buf: &mut [u8]) -> io::Result<usize>
```


<a id="c-overload"></a>
## Operator overloads are unsurprising (C-OVERLOAD)

Operators with built in syntax (`*`, `|`, and so on) can be provided for a type
by implementing the traits in [`std::ops`]. These operators come with strong
expectations: implement `Mul` only for an operation that bears some resemblance
to multiplication (and shares the expected properties, e.g. associativity), and
so on for the other traits.

[`std::ops`]: https://doc.rust-lang.org/std/ops/index.html#traits


<a id="c-deref"></a>
## `Deref` and `DerefMut` are unsurprising (C-DEREF)

The `Deref` traits are used implicitly by the compiler in many circumstances,
and interact with method resolution. The relevant rules are designed
specifically to accommodate smart pointers, and so the traits should be used
with those rules in mind.

As a general guideline, consider the "_as-a_" rule. If you implement `Deref`,
wherever `&Target` is acceptable, `&Self` works just as well as `&Target`.

### Examples from the standard library

- [`Box<T>`](https://doc.rust-lang.org/std/boxed/struct.Box.html)
- [`String`](https://doc.rust-lang.org/std/string/struct.String.html) is a smart
  pointer to [`str`](https://doc.rust-lang.org/std/primitive.str.html)
- [`Rc<T>`](https://doc.rust-lang.org/std/rc/struct.Rc.html)
- [`Arc<T>`](https://doc.rust-lang.org/std/sync/struct.Arc.html)
- [`Cow<'a, T>`](https://doc.rust-lang.org/std/borrow/enum.Cow.html)
- [`ManuallyDrop<T>`](https://doc.rust-lang.org/stable/std/mem/struct.ManuallyDrop.html) (not a traditional pointer)
- [`AssertUnwindSafe<T>`](https://doc.rust-lang.org/stable/std/panic/struct.AssertUnwindSafe.html) (not a traditional pointer)


<a id="c-ctor"></a>
## Constructors are static, inherent methods (C-CTOR)

In Rust, "constructors" are just a convention. There are a variety of
conventions around constructor naming, and the distinctions are often
subtle.

A constructor in its most basic form is a `new` method with no arguments.

```rust
impl<T> Example<T> {
    pub fn new() -> Example<T> { /* ... */ }
}
```

Constructors are static (no `self`) inherent methods for the type that they
construct. Combined with the practice of fully importing type names, this
convention leads to informative but concise construction:

```rust
use example::Example;

// Construct a new Example.
let ex = Example::new();
```

The name `new` should generally be used for the primary method of instantiating
a type. Sometimes it takes no arguments, as in the examples above. Sometimes it
does take arguments, like [`Box::new`] which is passed the value to place in the
`Box`.

Some types' constructors, most notably I/O resource types, use distinct naming
conventions for their constructors, as in [`File::open`], [`Mmap::open`],
[`TcpStream::connect`], and [`UdpSocket::bind`]. In these cases names are chosen
as appropriate for the domain.

Often there are multiple ways to construct a type. It's common in these cases
for secondary constructors to be suffixed `_with_foo`, as in
[`Mmap::open_with_offset`]. If your type has a multiplicity of construction
options though, consider the builder pattern ([C-BUILDER]) instead.

Some constructors are "conversion constructors", methods that create a new type
from an existing value of a different type. These typically have names beginning
with `from_` as in [`std::io::Error::from_raw_os_error`]. Note also though the
`From` trait ([C-CONV-TRAITS]), which is quite similar. There are three
distinctions between a `from_`-prefixed conversion constructor and a `From<T>`
impl.

- A `from_` constructor can be unsafe; a `From` impl cannot. One example of this
  is [`Box::from_raw`].
- A `from_` constructor can accept additional arguments to disambiguate the
  meaning of the source data, as in [`u64::from_str_radix`].
- A `From` impl is only appropriate when the source data type is sufficient to
  determine the encoding of the output data type. When the input is just a bag
  of bits like in [`u64::from_be`] or [`String::from_utf8`], the conversion
  constructor name is able to identify their meaning.

[`Box::from_raw`]: https://doc.rust-lang.org/std/boxed/struct.Box.html#method.from_raw
[`u64::from_str_radix`]: https://doc.rust-lang.org/std/primitive.u64.html#method.from_str_radix
[`u64::from_be`]: https://doc.rust-lang.org/std/primitive.u64.html#method.from_be
[`String::from_utf8`]: https://doc.rust-lang.org/std/string/struct.String.html#method.from_utf8

Note that it is common and expected for types to implement both `Default` and a
`new` constructor. For types that have both, they should have the same behavior.
Either one may be implemented in terms of the other.

[C-BUILDER]: type-safety.html#c-builder
[C-CONV-TRAITS]: interoperability.html#c-conv-traits

### Examples from the standard library

- [`std::io::Error::new`] is the commonly used constructor for an IO error.
- [`std::io::Error::from_raw_os_error`] is a conversion constructor
  based on an error code received from the operating system.
- [`Box::new`] creates a new container type, taking a single argument.
- [`File::open`] opens a file resource.
- [`Mmap::open_with_offset`] opens a memory-mapped file, with additional options.

[`File::open`]: https://doc.rust-lang.org/stable/std/fs/struct.File.html#method.open
[`Mmap::open`]: https://docs.rs/memmap/0.5.2/memmap/struct.Mmap.html#method.open
[`Mmap::open_with_offset`]: https://docs.rs/memmap/0.5.2/memmap/struct.Mmap.html#method.open_with_offset
[`TcpStream::connect`]: https://doc.rust-lang.org/stable/std/net/struct.TcpStream.html#method.connect
[`UdpSocket::bind`]: https://doc.rust-lang.org/stable/std/net/struct.UdpSocket.html#method.bind
[`std::io::Error::new`]: https://doc.rust-lang.org/std/io/struct.Error.html#method.new
[`std::io::Error::from_raw_os_error`]: https://doc.rust-lang.org/std/io/struct.Error.html#method.from_raw_os_error
[`Box::new`]: https://doc.rust-lang.org/stable/std/boxed/struct.Box.html#method.new

