% Braces, semicolons, and commas

### Opening braces always go on the same line. [RFC]

``` rust
fn foo() {
    ...
}

fn frobnicate(a: Bar, b: Bar,
              c: Bar, d: Bar)
              -> Bar {
    ...
}

trait Bar {
    fn baz(&self);
}

impl Bar for Baz {
    fn baz(&self) {
        ...
    }
}

frob(|x| {
    x.transpose()
})
```

### `match` arms get braces, except for single-line expressions. [RFC]

``` rust
match foo {
    bar => baz,
    quux => {
        do_something();
        do_something_else()
    }
}
```

### `return` statements get semicolons. [RFC]

``` rust
fn foo() {
    do_something();

    if condition() {
        return;
    }

    do_something_else();
}
```

### Trailing commas [OPEN]

> **[OPEN]** We should have a guideline for when to include trailing
> commas in `struct`s, `match`es, function calls, etc.
>
> One possible rule: a trailing comma should be included whenever the
> closing delimiter appears on a separate line:

```rust
Foo { bar: 0, baz: 1 }

Foo {
    bar: 0,
    baz: 1,
}

match a_thing {
    None => 0,
    Some(x) => 1,
}
```
