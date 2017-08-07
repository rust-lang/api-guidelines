# Flexibility


<a id="c-intermediate"></a>
## Functions expose intermediate results to avoid duplicate work (C-INTERMEDIATE)

Many functions that answer a question also compute interesting related data. If
this data is potentially of interest to the client, consider exposing it in the
API.

### Examples from the standard library

- [`Vec::binary_search`] does not return a `bool` of whether the value was
  found, nor an `Option<usize>` of the index at which the value was maybe found.
  Instead it returns information about the index if found, and also the index at
  which the value would need to be inserted if not found.

- [`String::from_utf8`] may fail if the input bytes are not UTF-8. In the error
  case it returns an intermediate result that exposes the byte offset up to
  which the input was valid UTF-8, as well as handing back ownership of the
  input bytes.

- [`HashMap::insert`] returns an `Option<T>` that returns the preexisting value
  for a given key, if any. For cases where the user wants to recover this value
  having it returned by the insert operation avoids the user having to do a second
  hash table lookup.

[`Vec::binary_search`]: https://doc.rust-lang.org/std/vec/struct.Vec.html#method.binary_search
[`String::from_utf8`]: https://doc.rust-lang.org/std/string/struct.String.html#method.from_utf8
[`HashMap::insert`]: https://doc.rust-lang.org/stable/std/collections/struct.HashMap.html#method.insert


<a id="c-caller-control"></a>
## Caller decides where to copy and place data (C-CALLER-CONTROL)

If a function requires ownership of an argument, it should take ownership of the
argument rather than borrowing and cloning the argument.

```rust
// Prefer this:
fn foo(b: Bar) {
    /* use b as owned, directly */
}

// Over this:
fn foo(b: &Bar) {
    let b = b.clone();
    /* use b as owned after cloning */
}
```

If a function *does not* require ownership of an argument, it should take a
shared or exclusive borrow of the argument rather than taking ownership and
dropping the argument.

```rust
// Prefer this:
fn foo(b: &Bar) {
    /* use b as borrowed */
}

// Over this:
fn foo(b: Bar) {
    /* use b as borrowed, it is implicitly dropped before function returns */
}
```

The `Copy` trait should only be used as a bound when absolutely needed, not as a
way of signaling that copies should be cheap to make.


<a id="c-generic"></a>
## Functions minimize assumptions about parameters by using generics (C-GENERIC)

The fewer assumptions a function makes about its inputs, the more widely usable
it becomes.

Prefer

```rust
fn foo<I: IntoIterator<Item = i64>>(iter: I) { /* ... */ }
```

over any of

```rust
fn foo(c: &[i64]) { /* ... */ }
fn foo(c: &Vec<i64>) { /* ... */ }
fn foo(c: &SomeOtherCollection<i64>) { /* ... */ }
```

if the function only needs to iterate over the data.

More generally, consider using generics to pinpoint the assumptions a function
needs to make about its arguments.

### Advantages of generics

* _Reusability_. Generic functions can be applied to an open-ended collection of
  types, while giving a clear contract for the functionality those types must
  provide.

* _Static dispatch and optimization_. Each use of a generic function is
  specialized ("monomorphized") to the particular types implementing the trait
  bounds, which means that (1) invocations of trait methods are static, direct
  calls to the implementation and (2) the compiler can inline and otherwise
  optimize these calls.

* _Inline layout_. If a `struct` and `enum` type is generic over some type
  parameter `T`, values of type `T` will be laid out inline in the
  `struct`/`enum`, without any indirection.

* _Inference_. Since the type parameters to generic functions can usually be
  inferred, generic functions can help cut down on verbosity in code where
  explicit conversions or other method calls would usually be necessary.

* _Precise types_. Because generic give a _name_ to the specific type
  implementing a trait, it is possible to be precise about places where that
  exact type is required or produced. For example, a function

  ```rust
  fn binary<T: Trait>(x: T, y: T) -> T
  ```

  is guaranteed to consume and produce elements of exactly the same type `T`; it
  cannot be invoked with parameters of different types that both implement
  `Trait`.

### Disadvantages of generics

* _Code size_. Specializing generic functions means that the function body is
  duplicated. The increase in code size must be weighed against the performance
  benefits of static dispatch.

* _Homogeneous types_. This is the other side of the "precise types" coin: if
  `T` is a type parameter, it stands for a _single_ actual type. So for example
  a `Vec<T>` contains elements of a single concrete type (and, indeed, the
  vector representation is specialized to lay these out in line). Sometimes
  heterogeneous collections are useful; see [trait objects][C-OBJECT].

* _Signature verbosity_. Heavy use of generics can make it more difficult to
  read and understand a function's signature.

### Examples from the standard library

- [`std::fs::File::open`] takes an argument of generic type `AsRef<Path>`. This
  allows files to be opened conveniently from a string literal `"f.txt"`, a
  [`Path`], an [`OsString`], and a few other types.

[`std::fs::File::open`]: https://doc.rust-lang.org/std/fs/struct.File.html#method.open
[`Path`]: https://doc.rust-lang.org/std/path/struct.Path.html
[`OsString`]: https://doc.rust-lang.org/std/ffi/struct.OsString.html


<a id="c-object"></a>
## Traits are object-safe if they may be useful as a trait object (C-OBJECT)

Trait objects have some significant limitations: methods invoked through a trait
object cannot use generics, and cannot use `Self` except in receiver position.

When designing a trait, decide early on whether the trait will be used as an
object or as a bound on generics.

If a trait is meant to be used as an object, its methods should take and return
trait objects rather than use generics.

A `where` clause of `Self: Sized` may be used to exclude specific methods from
the trait's object. The following trait is not object-safe due to the generic
method.

```rust
trait MyTrait {
    fn object_safe(&self, i: i32);

    fn not_object_safe<T>(&self, t: T);
}
```

Adding a requirement of `Self: Sized` to the generic method excludes it from the
trait object and makes the trait object-safe.

```rust
trait MyTrait {
    fn object_safe(&self, i: i32);

    fn not_object_safe<T>(&self, t: T) where Self: Sized;
}
```

### Advantages of trait objects

* _Heterogeneity_. When you need it, you really need it.
* _Code size_. Unlike generics, trait objects do not generate specialized
  (monomorphized) versions of code, which can greatly reduce code size.

### Disadvantages of trait objects

* _No generic methods_. Trait objects cannot currently provide generic methods.
* _Dynamic dispatch and fat pointers_. Trait objects inherently involve
  indirection and vtable dispatch, which can carry a performance penalty.
* _No Self_. Except for the method receiver argument, methods on trait objects
  cannot use the `Self` type.

### Examples from the standard library

- The [`io::Read`] and [`io::Write`] traits are often used as objects.
- The [`Iterator`] trait has several generic methods marked with `where Self:
  Sized` to retain the ability to use `Iterator` as an object.

[`io::Read`]: https://doc.rust-lang.org/std/io/trait.Read.html
[`io::Write`]: https://doc.rust-lang.org/std/io/trait.Write.html
[`Iterator`]: https://doc.rust-lang.org/std/iter/trait.Iterator.html


<a id="c-struct-bounds"></a>
## Data structures do not have trait bounds (C-STRUCT-BOUNDS)

Use trait bounds on `impl` blocks for behavior, rather than on data structures.

```rust
// Prefer this:
struct MyReader<R> {
    inner: R,
}

// Over this:
struct MyReader<R: Read> {
    inner: R,
}
```

Trait bounds on data structures are virally required by other structures that contain them.
They're also required by all `impl` blocks, whether or not that bound is actually used
within the block.

Some structures are only ever intended to contain values that implement a particular trait.
In this case, prefer trait bounds on an `impl` block rather than the structure itself:

```rust
// Prefer this:
struct MyReader<R> {
    inner: R,
}

impl<R: Read> MyReader<R> {
  fn new(reader: R) -> Self { /* ... */ }
}

// Over this:
struct MyReader<R: Read> {
    inner: R,
}

impl<R: Read> MyReader<R> {
  fn new(reader: R) -> Self { /* ... */ }
}
```

### Exceptions

There are three exceptions where trait bounds on structures are required:

1. The data structure refers to an associated type on the trait
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
