# Rust Guidelines [working title]

This document collects the emerging principles, conventions, abstractions, and
best practices for writing Rust code.

Since Rust is evolving at a rapid pace, these guidelines are very
preliminary. The hope is that writing them down explicitly will help drive
discussion, consensus and adoption.

Whenever feasible, guidelines provide specific examples from Rust's standard
libraries.

### Building the document

Like http://rustbyexample.com/, this guidelines document is written as
a [gitbook](https://github.com/GitbookIO/gitbook). You can install `gitbook` by running

```
npm install gitbook -g
```

After installing, just run

```
gitbook serve
```

in the root of the repository to continuously serve your local copy of
the guidelines.

### Guideline statuses

Every guideline has a status:

* **[FIXME]**: Marks places where there is clear work to be done that does not
  require further discussion or consensus.

* **[OPEN]**: Marks open questions that require concrete proposals for further
  discussion.

* **[RFC]**: Marks concrete, proposed guidelines that need further discussion to
  reach consensus.

* Untagged guidelines are considered well-accepted. _**Ed. note**: to
  begin with, there are almost none of these!_

#### Pre-draft stage

To begin with, this document will exist under
https://github.com/aturon/rust-guidelines and will have no official status.
This brief early stage is mainly intended as a sanity check, and to try to get
the draft in as good of shape as possible before bringing it to the full community.

_Pull requests very welcome!_

#### First draft stage

After completing a first draft of the guidelines, we will:

1. Move the repository into the `rust-lang` organization, and
2. Begin a formal process to migrate guidelines to fully accepted status.

The precise mechanics for guideline adoption are to be determined, but will
likely resemble the Rust RFC process.

### What's in this document

This document is broken into four parts:

* **[Style](style/README.md)** provides a set of rules governing naming conventions,
  whitespace, and other stylistic issues.

* **[Guidelines by Rust feature](features/README.md)** places the focus on each of
  Rust's features, starting from expressions and working the way out toward
  crates, dispensing guidelines relevant to each.

* **Topical guidelines and patterns**. The rest of the document proceeds by
  cross-cutting topic, starting with
  [Ownership and resources](ownership/README.md).

* **[APIs for a changing Rust](change/README.md)**
  discusses the forward-compatibility hazards, especially those that interact
  with the pre-1.0 library stabilization process.

> **[FIXME]** Add cross-references throughout this document to the tutorial,
> reference manual, and other guides.

> **[OPEN]** What are some _non_-goals, _non_-principles, or _anti_-patterns that
> we should document?
