% Naming conventions

### General conventions

| Item | Convention |
| ---- | ---------- |
| Crates | `snake_case` (but prefer single word) |
| Modules | `snake_case` |
| Types | `CamelCase` |
| Traits | `CamelCase`; prefer transitive verbs, nouns, and then adjectives; avoid suffixes (like `able`) |
| Enum variants | `CamelCase` |
| Functions | `snake_case` |
| Conversions | `as_foo`/`to_foo`/`into_foo` (see [subsection](containers.md)) |
| Methods | `snake_case` |
| General constructors | `new` or `new_with_more_details` |
| Conversion constructors | `from_some_other_type` |
| Local variables | `snake_case` |
| Static variables | `SCREAMING_SNAKE_CASE` |
| Type parameters | single uppercase letter: `T` |
| Lifetimes | short, lowercase: `'a` |

<p>
In `CamelCase`, acronyms count as one word: use `Uuid` rather than `UUID`.

### Referring to types in function/method names [FIXME]

<!-- `&T` -> `ref` -->
<!-- `&mut T` -> `mut` -->

> **[FIXME]** We should establish general conventions for type names
> when they appear as part of functions/methods. For example, that
> `&[U]` is generally called `slice`.

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


### Fallible functions [FIXME]

> **[FIXME]** Should we have a standard marker for functions that can
> cause task failure?

> See https://github.com/rust-lang/rust/issues/13159

### Getter/setter methods [FIXME]

> **[FIXME]** Need a naming and signature convention here.

### Escape hatches [FIXME]

> **[FIXME]** Should we standardize a convention for functions that may break API
> guarantees? e.g. `ToCStr::to_c_str_unchecked`

### Predicates

* Simple boolean predicates should be prefixed with `is_` or another
  short question word, e.g., `is_empty`.
* Common exceptions: `lt`, `gt`, and other established predicate names.
