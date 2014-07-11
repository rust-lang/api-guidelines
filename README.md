# Rust Guidelines [working title]

This document collects the emerging principles, conventions, abstractions, and
best practices for writing Rust code.

Since Rust is evolving at a rapid pace, these guidelines are
preliminary. The hope is that writing them down explicitly will help
drive discussion, consensus and adoption.

Whenever feasible, guidelines provide specific examples from Rust's standard
libraries.

For now, you can find a rendered snapshot at
[http://aturon.github.io/](http://aturon.github.io/).  After
[some infrastructure work](https://github.com/aturon/rust-guidelines/issues/17), the snapshot will move somewhere more
official.

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

### Guideline stabilization

One purpose of these guidelines is to reach decisions on a number of
cross-cutting API and stylistic choices. Discussion and development of the
guidelines will happen through:

1. **Primarily**: http://discuss.rust-lang.org/, using the Guidelines category.
2. On the github issue tracker.
3. Pull requests to add or change guidelines.

Guidelines that are under development or discussion will be marked with the
status **[RFC]** (or **[OPEN]** in the very early stages).

The Rust team will hold meetings to move guidelines from **[RFC]** to approved
(untagged) status, similar to the current practice for RFCs.

The specifics of the process will evolve over time, and approved guidelines may
still change.

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

* **[APIs for a changing Rust](changing/README.md)**
  discusses the forward-compatibility hazards, especially those that interact
  with the pre-1.0 library stabilization process.

> **[FIXME]** Add cross-references throughout this document to the tutorial,
> reference manual, and other guides.

> **[OPEN]** What are some _non_-goals, _non_-principles, or _anti_-patterns that
> we should document?
