# Signaling errors [RFC]

### For unrecoverable errors, `fail!`.

Unrecoverable errors are rare and catastrophic.  Since there is no way
to cleanly exit the operation that discovered the error, there is no
sensible return value. Instead, `fail` unwinds the task's stack. See
"Handling Errors" below for guidance on how to use tasks to isolate
unrecoverable errors.

For example,
[`rt::heap::allocate`](http://static.rust-lang.org/doc/master/std/rt/heap/fn.allocate.html)
fails the task if allocation fails.

### For recoverable but unavoidable errors, use `Result`.

The `E` component in `Result<T,E>` should convey details about the external
circumstances that caused the error. Use an `enum` when multiple kinds of errors
are possible.

For example, the
[`std::io`](http://static.rust-lang.org/doc/master/std/io/index.html) module
uses the `Result` type pervasively, with
[`IoError`](http://static.rust-lang.org/doc/master/std/io/struct.IoError.html)
providing details when an error occurs.

### For recoverable and avoidable errors, prefer `Result`.

Avoidable errors can only arise when a function places additional expectations
on its input beyond its static type. When the error is also recoverable, there
are two choices.

#### Preferred: returning a `Result`

Following the principle of
[returning useful intermediate results](../features/functions-and-methods/output.md),
the function should return a `Result` value with an error component saying
exactly where things went wrong. This information is typically produced when
validating or consuming input anyway; returning it to the client allows for
better error reporting and possibly recovery.

For example, a function `from_str` for parsing into a given type should return a
`Result` with error component giving the location or span of parsing
failure.

Clients can treat the function's contract as a kind of assertion by
calling `unwrap` on the `Result`, which will produce a failure _in the
client's code_ when the client breaks the contract.

#### Failing.

While returning a `Result` makes for a very clear interface, it can be
burdensome:

* Clients have to `unwrap` the result to assert that they have
  satisfied the function's contract.
* The function has to internally propagate the `Result`, and possibly
  perform extra cleanup.

Sometimes avoidable, recoverable errors are very rare and clearly
indicate a logic error rather than bad input. Such errors are also
likely to cause task failure at some point (usually because clients
would just `unwrap`).

In those rare cases, it is acceptable to invoke `fail!` when the error
is discovered. The expectations on the input then become a strong part
of the function's contract, and violating them is always a preventable
bug. The failure conditions should be clearly stated in a `Failure`
section of the function's doc comment.

For example, slice indexing fails on an out of bounds error, which is
recoverable and avoidable but nearly always represents a bug (and is
hopefully rare).

> **[FIXME]** `std` also contains some more dubious examples, like the
> [`to_c_str`](http://static.rust-lang.org/doc/master/std/c_str/trait.ToCStr.html#tymethod.to_c_str)
> method, which fails when the string contains an interior
> `null`. What do we want to say about those?

### Fallible convenience methods. [OPEN]

> **[OPEN]** This is a big open issue: when should APIs provide
> convenience methods that amount to `unwrap`ping a `Result`? E.g.,
> the `RefCell` type has `borrow` and `try_borrow`, with `.borrow()`
> equivalent to `try_borrow().unwrap()`.

### Avoid `Option` for error signaling.

The `Option` type should be reserved for cases that do not represent errors, but
rather possible outcomes for well-formed inputs under normal circumstances.

For example,
[the `Vec::pop` method](http://static.rust-lang.org/doc/master/std/vec/struct.Vec.html)
returns an `Option`, yielding `None` when the vector is empty.

Even when there is no interesting error information to return, prefer `Result<T,
()>` to `Option<T>` as a way of signaling that the `Err` case represents an
erroneous outcome, not a normal outcome.

Rather than providing a `try_frob` function yielding an `Option`, provide a
`frob` function yielding a `Result`.
