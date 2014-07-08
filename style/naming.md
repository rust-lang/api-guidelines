# Naming conventions

### General conventions [RFC]

| Item | Convention |
| ---- | ---------- |
| Crates | `snake_case` (but prefer single word) |
| Modules | `snake_case` |
| Types | `CamelCase` |
| Traits | `CamelCase`; prefer transitive verbs, nouns, and then adjectives; avoid suffixes (like `able`) |
| Enum variants | `CamelCase` |
| Functions | `snake_case` |
| Conversions | `as_foo`/`to_foo`/`into_foo` (see below) |
| Methods | `snake_case` |
| General constructors | `new` or `new_with_more_details` |
| Conversion constructors | `from_some_other_type` |
| Local variables | `snake_case` |
| Static variables | `SCREAMING_SNAKE_CASE` |
| Type parameters | single uppercase letter: `T` |
| Lifetimes | short, lowercase: `'a` |

<p>
In `CamelCase`, acronyms count as one word: use `Uuid` rather than `UUID`.

> **[FIXME]** The convention for conversion constructors is at odds
> with the convention for type names. Should a conversion from
> `HashMap` be called `from_hash_map` or `from_HashMap`?

### Avoid redundant prefixes [RFC]

Names of items within a module should not be prefixed with that module's name,
since clients can always reference the item qualified by the module name.

Prefer

``` rust
mod foo {
    pub struct Bar { ... }
}
```

over

``` rust
mod foo {
    pub struct FooBar { ... }
}
```

### Ownership variants [OPEN]

> **[OPEN]** What about ownership-related variants of functions? e.g. `move_`
> and `mut_` prefixes (or suffixes)? See
> https://github.com/mozilla/rust/issues/13660 for example.

### Fallible functions [OPEN]

> **[OPEN]** Should we have a standard marker for functions that can
> cause task failure?

> See https://github.com/mozilla/rust/issues/13159

### Unwrapping, extracting, taking [OPEN]

> **[OPEN]** We need to standardize names for methods that
> extract/swap/remove from boxes/options/other singleton containers
> and smart pointers

> See https://github.com/mozilla/rust/issues/13159

Also related to fallible functions, above.

### Getter/setter methods [OPEN]

> **[OPEN]** Need a naming and signature convention here.

### Escape hatches [OPEN]

> **[OPEN]** Should we standardize a convention for functions that may break API
> guarantees? e.g. `ToCStr::to_c_str_unchecked`

### Conversions [RFC]

> See https://github.com/mozilla/rust/issues/7087

> **[OPEN]** Should we provide standard traits for conversions? Doing
> so nicely will require
> [trait reform](https://github.com/rust-lang/rfcs/pull/48) to land.

Conversions should be provided as methods, with names prefixed as follows:

| Prefix | Cost | Consumes convertee |
| ------ | ---- | ------------------ |
| `as_` | Free | No |
| `to_` | Expensive | No |
| `into_` | Variable | Yes |

<p>
For example:

* `as_bytes()` gives a `&[u8]` view into a `&str`, which is a no-op.
* `to_owned()` copies a `&str` to a new `String`.
* `into_bytes()` consumes a `String` and yields the underlying
  `Vec<u8>`, which is a no-op.

Conversions prefixed `as_` and `into_` typically _decrease abstraction_, either
exposing a view into the underlying representation (`as`) or deconstructing data
into its underlying representation (`into`). Conversions prefixed `to_`, on the
other hand, typically stay at the same level of abstraction but do some work to
change one representation into another.

> **[OPEN]** The distinctions between conversion methods does not work
> so well for `from_` conversion constructors. Is that a problem?

### Iterators

> **[FIXME]** Need to add guidance on iterator function names, e.g. `iter`,
> `move_iter`, `ref_iter`. Related to the `Iterable` proposal
> (https://github.com/rust-lang/rfcs/pull/17/).

> See [PR #11001](https://github.com/mozilla/rust/pull/11001).

> **[OPEN]** Is the `Move` prefix a relic of an older rust?

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

### Predicates

* Simple boolean predicates should be prefixed with `is_` or another
  short question word, e.g., `is_empty`.
* Common exceptions: `lt`, `gt`, and other established predicate names.
