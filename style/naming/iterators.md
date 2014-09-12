% Iterators

#### Method names

For a container with elements of type `U`, iterator methods should be named:

```rust
fn iter(&self) -> T           // where T implements Iterator<&U>
fn iter_mut(&mut self) -> T   // where T implements Iterator<&mut U>
fn into_iter(self) -> T       // where T implements Iterator<U>
```

The default iterator variant yields shared references `&U`.

#### Type names [RFC - awaiting revision]

Iterators require introducing and exporting new types. These types should use
the following naming convention:

* **Base name**. If the iterator yields something that can be described with a
   specific noun, the base name should be the pluralization of that noun
   (e.g. an iterator yielding words is called `Words`). Generic contains use the
   base name `Items`.

* **Flavor prefix**. Iterators often come in multiple flavors, with the default
  flavor providing immutable references. Other flavors should prefix their name:

  * Moving iterators have a prefix of `Move`.
  * If the default iterator yields an immutable reference, an iterator
    yielding a mutable reference has a prefix `Mut`.
  * Reverse iterators have a prefix of `Rev`.
