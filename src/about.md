# Rust API Guidelines

This is a set of recommendations on how to design and present APIs for the Rust
programming language. They are authored largely by the Rust library team, based
on experiences building the Rust standard library and other crates in the Rust
ecosystem.

These are only guidelines, some more firm than others. In some cases they are
vague and still in development. Rust crate authors should consider them as a set
of important considerations in the development of idiomatic and interoperable
Rust libraries, to use as they see fit. These guidelines should not in any way
be considered a mandate that crate authors must follow, though they may find
that crates that conform well to these guidelines integrate better with the
existing crate ecosystem than those that do not.

This book is organized in two parts: the concise [checklist] of all individual
guidelines, suitable for quick scanning during crate reviews; and topical
chapters containing explanations of the guidelines in detail.

If you are interested in contributing to the API guidelines, check out
[contributing.md] and join our [Gitter channel].

[checklist]: checklist.html
[contributing.md]: https://github.com/rust-lang/api-guidelines/blob/master/CONTRIBUTING.md
[Gitter channel]: https://gitter.im/rust-impl-period/WG-libs-guidelines
