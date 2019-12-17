# Contributing to the API guidelines

The Rust API guidelines project welcomes contribution from everyone in the form
of suggestions, bug reports, pull requests, and feedback. This document gives
some guidance if you are thinking of helping us.

Please reach out here in a GitHub issue or in our [Gitter channel] if we can do
anything to help you contribute.

[Gitter channel]: https://gitter.im/rust-impl-period/WG-libs-guidelines

## Submitting ideas for new guidelines

We are always looking out for lessons to learn from high-quality Rust libraries.
If you spot an aspect of some crate's API that you think other crates could
benefit from, please [file an issue] to let us know.

[file an issue]: https://github.com/rust-lang/api-guidelines/issues/new

## Writing content for the guidelines

The guidelines are written in a collection of Markdown files under the [`src`]
directory. When making changes, you can preview the rendered content using
[mdBook].

[`src`]: https://github.com/rust-lang/api-guidelines/tree/master/src
[mdBook]: https://github.com/azerupi/mdBook

```sh
cargo install mdbook

# In the api-guidelines directory
mdbook serve
```

The `mdbook serve` command makes the rendered API guidelines available as a web
page at http://localhost:3000/.

## Guidelines for the guidelines

We follow some basic grammatical rules to ensure that the checklist of
guidelines remains consistent and intelligible.

A guideline is an **indicative statement** about a hypothetical crate.

```diff
- Not an imperative like "Implement Hex, Octal, Binary for binary number types"
+ Instead: "Binary number types provide Hex, Octal, Binary formatting"

- Not an obligation like "Macros should compose well with attributes"
+ Instead: "Macros compose well with attributes"
```

Guidelines have an explicit **subject** and **verb**.

```diff
- Not implicit subject like "Includes all common Cargo.toml metadata"
+ Instead: "Cargo.toml includes all common metadata"

- Not implicit verb like "Thoroughly documented with examples"
+ Instead: "Crate level docs are thorough and include examples"

- Not metaphysical like "There are no out-parameters"
+ Instead: "Functions do not take out-parameters"
```

Guidelines use **active voice**.

```diff
- Not passive voice like "Function arguments are validated"
+ Instead: "Functions validate their arguments"
```

## Conduct

As with all Rust-related spaces, we observe the [Rust Code of Conduct]. For
escalation or moderation issues please contact the Rust moderation team,
rust-mods@rust-lang.org.

[Rust Code of Conduct]: https://www.rust-lang.org/conduct.html
