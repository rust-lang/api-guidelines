# Macros


<a id="c-evocative"></a>
## Input syntax is evocative of the output (C-EVOCATIVE)

Rust macros let you dream up practically whatever input syntax you want. Aim to
keep input syntax familiar and cohesive with the rest of your users' code by
mirroring existing Rust syntax where possible. Pay attention to the choice and
placement of keywords and punctuation.

A good guide is to use syntax, especially keywords and punctuation, that is
similar to what will be produced in the output of the macro.

For example if your macro declares a struct with a particular name given in the
input, preface the name with the keyword `struct` to signal to readers that a
struct is being declared with the given name.

```rust
// Prefer this...
bitflags! {
    struct S: u32 { /* ... */ }
}

// ...over no keyword...
bitflags! {
    S: u32 { /* ... */ }
}

// ...or some ad-hoc word.
bitflags! {
    flags S: u32 { /* ... */ }
}
```

Another example is semicolons vs commas. Constants in Rust are followed by
semicolons so if your macro declares a chain of constants, they should likely be
followed by semicolons even if the syntax is otherwise slightly different from
Rust's.

```rust
// Ordinary constants use semicolons.
const A: u32 = 0b000001;
const B: u32 = 0b000010;

// So prefer this...
bitflags! {
    struct S: u32 {
        const C = 0b000100;
        const D = 0b001000;
    }
}

// ...over this.
bitflags! {
    struct S: u32 {
        const E = 0b010000,
        const F = 0b100000,
    }
}
```

Macros are so diverse that these specific examples won't be relevant, but think
about how to apply the same principles to your situation.


<a id="c-macro-attr"></a>
## Item macros compose well with attributes (C-MACRO-ATTR)

Macros that produce more than one output item should support adding attributes
to any one of those items. One common use case would be putting individual items
behind a cfg.

```rust
bitflags! {
    struct Flags: u8 {
        #[cfg(windows)]
        const ControlCenter = 0b001;
        #[cfg(unix)]
        const Terminal = 0b010;
    }
}
```

Macros that produce a struct or enum as output should support attributes so that
the output can be used with derive.

```rust
bitflags! {
    #[derive(Default, Serialize)]
    struct Flags: u8 {
        const ControlCenter = 0b001;
        const Terminal = 0b010;
    }
}
```


<a id="c-anywhere"></a>
## Item macros work anywhere that items are allowed (C-ANYWHERE)

Rust allows items to be placed at the module level or within a tighter scope
like a function. Item macros should work equally well as ordinary items in all
of these places. The test suite should include invocations of the macro in at
least the module scope and function scope.

```rust
#[cfg(test)]
mod tests {
    test_your_macro_in_a!(module);

    #[test]
    fn anywhere() {
        test_your_macro_in_a!(function);
    }
}
```

As a simple example of how things can go wrong, this macro works great in a
module scope but fails in a function scope.

```rust
macro_rules! broken {
    ($m:ident :: $t:ident) => {
        pub struct $t;
        pub mod $m {
            pub use super::$t;
        }
    }
}

broken!(m::T); // okay, expands to T and m::T

fn g() {
    broken!(m::U); // fails to compile, super::U refers to the containing module not g
}
```


<a id="c-macro-vis"></a>
## Item macros support visibility specifiers (C-MACRO-VIS)

Follow Rust syntax for visibility of items produced by a macro. Private by
default, public if `pub` is specified.

```rust
bitflags! {
    struct PrivateFlags: u8 {
        const A = 0b0001;
        const B = 0b0010;
    }
}

bitflags! {
    pub struct PublicFlags: u8 {
        const C = 0b0100;
        const D = 0b1000;
    }
}
```


<a id="c-macro-ty"></a>
## Type fragments are flexible (C-MACRO-TY)

If your macro accepts a type fragment like `$t:ty` in the input, it should be
usable with all of the following:

- Primitives: `u8`, `&str`
- Relative paths: `m::Data`
- Absolute paths: `::base::Data`
- Upward relative paths: `super::Data`
- Generics: `Vec<String>`

As a simple example of how things can go wrong, this macro works great with
primitives and absolute paths but fails with relative paths.

```rust
macro_rules! broken {
    ($m:ident => $t:ty) => {
        pub mod $m {
            pub struct Wrapper($t);
        }
    }
}

broken!(a => u8); // okay

broken!(b => ::std::marker::PhantomData<()>); // okay

struct S;
broken!(c => S); // fails to compile
```
