% Signaling errors [RFC]

Errors fall into one of three categories:

* Catastrophic errors, e.g. out of memory.
* Unpreventable errors, e.g. file not found.
* Preventable errors, e.g. wrong input encoding, index out of bounds.

The signaling strategy is determined by the error category.

## Catastrophic errors

An error is _catastrophic_ if there is no meaningful way for the current task to
continue after the error occurs.

Catastrophic errors are _extremely_ rare, especially outside of `libstd`.

**Canonical examples**: out of memory, stack overflow.

> **[FIXME]** These are believed to be the _only_ catastrophic errors in
> Rust/libstd at the moment. We should confirm, and say so.

### For catastrophic errors, use `fail!`.

Invoking `fail!` causes the current task to fail, which:

* unwinds the task's stack
* poisons any channels or other synchronization connecting the task to others

For example,
[`rt::heap::allocate`](http://static.rust-lang.org/doc/master/std/rt/heap/fn.allocate.html)
fails the task if allocation fails.

Task failure does not mean that the entire program must abort.
For example, by breaking a program into a parent task that monitors its
children, the program _can_ meaningfully recover from task failures at the
global level. See [the error handling section](handling.md) for guidance
on working with task failure.

## Unpreventable errors

An error is _unpreventable_ if it is caused by circumstances out of the
program's control.

Unpreventable errors are common for operations that involve I/O or other access
to shared resources.

**Canonical example**: file not found.

### For unpreventable errors, use `Result`.

The
[`Result<T,E>` type](http://static.rust-lang.org/doc/master/std/result/index.html)
represents either a success (yielding `T`) or failure (yielding `E`). By
returning a `Result`, a function allows its clients to discover and react to
external circumstances that are obstructing an operation.

The `E` component in `Result<T,E>` should convey details about the external
circumstances that caused the error. Use an `enum` when multiple kinds of errors
are possible.

For example, the
[`std::io`](http://static.rust-lang.org/doc/master/std/io/index.html) module
uses the `Result` type pervasively, with
[`IoError`](http://static.rust-lang.org/doc/master/std/io/struct.IoError.html)
providing details when an error occurs.

## Preventable errors

An error is _preventable_ if its absence can be guaranteed by restricting the
arguments given to the operation.

Preventable errors are common for operations that impose constraints on their
parameters beyond their static type.

**Canonical examples**: wrong input encoding, index out of bounds.

### For preventable errors, prefer `Result`.

For preventable errors, API designers have to make a choice:
* Permit erroneous input, return `Result`, and use the `Err` variant to inform
  the client of the error.
* Treat erroneous input as a _contract violation_ (i.e., assertion failure) and `fail!`.

The right choice depends on the _overall design_ of an API: some APIs make it
very easy to obtain and work with valid inputs, in which case working with `Result` would be a needless distraction.

But when in doubt, prefer `Result`.

#### When to return a `Result`

For preventable errors in APIs where

* ensuring input validity ahead of time is difficult or expensive, or
* validity checking is a useful byproduct of attempting an operation,

return a `Result`.

Following the principle of
[returning useful intermediate results](../features/functions-and-methods/output.md),
the `Result`'s error component should give detail about which part of the input
was invalid -- information that is typically produced during input validation or
processing anyway.

For example, a function `from_str` for parsing into a given type should return a
`Result` with error component giving the location or span of parsing failure.

If clients believe they are providing valid input, they can use `unwrap` on the
`Result` to assert as much; this will produce a failure _in the client's
code_ if things go wrong.

#### When to `fail!`

For preventable errors in APIs where

* input validity can be easily checked, or
* parts of the API naturally produce valid inputs for other parts, or
* invalid inputs clearly represent a logic error,

using `fail!` is acceptable.

The expectations on the input then become a _contract_ for the function, and
violating them is akin to an assertion failure -- a bug. The failure conditions
should be clearly stated in a `Failure` section of the function's doc comment.

The benefit of using `fail!` is that the signatures of the API's functions are
simpler, and using the API does not require the client to constantly `unwrap`.

For example, slice indexing fails on an out of bounds error, which is a
preventable error and nearly always represents a bug. Moreover, most uses of
array indexing naturally incorporate a bounds check already, e.g. looping over
array indices.

> **[FIXME]** `std` also contains some more dubious examples, like the
> [`to_c_str`](http://static.rust-lang.org/doc/master/std/c_str/trait.ToCStr.html#tymethod.to_c_str)
> method, which fails when the string contains an interior
> `null`. What do we want to say about those?

#### Do not provide both `Result` and `fail!` variants. [RFC]

An API should not provide both `Result`-producing and `fail`ing versions of an
operation. It should provide just the `Result` version, allowing clients to use
`try!` or `unwrap` instead as needed.

There is one exception to this rule, however. Some APIs are strongly oriented
around failure, in the sense that their functions/methods are explicitly
intended as assertions.  If there is no other way to check in advance for the
validity of invoking an operation `foo`, however, the API may provide a
`checked_foo` variant that returns a `Result`.

The main examples in `libstd` providing both variants are:
* Channels, which are the primary point of failure propagation between tasks. As
  such, calling `recv()` is an _assertion_ that the other end of the channel is
  still alive, which will propagate failures from the other end of the
  channel. On the other hand, since there is no separate way to atomically test
  whether the other end has hung up, channels provide a `recv_opt` variant that
  produces a `Result`.

  > **[FIXME]** The `_opt` suffix needs to be replaced by a `checked_` prefix.


* `RefCell`, which provides a dynamic version of the borrowing rules. Calling
  the `borrow()` method is intended as an assertion that the cell is in a
  borrowable state, and will `fail!` otherwise. On the other hand, there is no
  separate way to check the state of the `RefCell`, so the module provides a
  `try_borrow` variant that produces a `Result`.

    > **[FIXME]** The `try_` prefix needs to be replaced by a `checked_` prefix.

### Avoid `Option` for error signaling. [RFC]

The `Option` type should be reserved for cases that do not represent errors, but
rather possible outcomes for well-formed inputs under normal circumstances.

For example,
[the `Vec::pop` method](http://static.rust-lang.org/doc/master/std/vec/struct.Vec.html)
returns an `Option`, yielding `None` when the vector is empty.

Even when there is no interesting error information to return, prefer
`Result<T,()>` to `Option<T>` as a way of signaling that the `Err` case represents an
erroneous outcome, not a normal outcome.

Rather than providing a `try_frob` function yielding an `Option`, provide a
`frob` function yielding a `Result`.
