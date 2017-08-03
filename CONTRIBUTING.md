# Guidelines for the guidelines

A guideline is an indicative statement about a hypothetical crate.

  - Not an imperative like "Implement Hex, Octal, Binary for binary number types."
    Instead: "Binary number types provide Hex, Octal, Binary formatting."

  - Not an obligation like "Macros should compose well with attributes."
    Instead: "Macros compose well with attributes."

Guidelines have an explicit subject and verb.

  - Not implicit subject like "Includes all common Cargo.toml metadata."
    Instead: "Cargo.toml includes all common metadata."

  - Not implicit verb like "Thoroughly documented with examples."
    Instead: "Crate level docs are thorough and include examples."

  - Not metaphysical like "There are no out-parameters."
    Instead: "Functions do not take out-parameters."

Guidelines use active voice.

  - Not passive voice like "Function arguments are validated."
    Instead: "Functions validate their arguments."

Links to other guidelines should use their C-IDENTIFIER to reinforce
the convention.

  - "But note that crates should re-export common functionality ([C-REEXPORT])."

[C-REEXPORT]: https://rust-lang-nursery.github.io/api-guidelines/organization.html#c-reexport
