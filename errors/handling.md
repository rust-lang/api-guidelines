# Handling errors

### Use task isolation to cope with failure. [FIXME]

> **[FIXME]** Explain how to isolate tasks and detect task failure for recovery.

### Never `unwrap` a `Result` for unavoidable errors. [RFC]

The `unwrap` method amounts to an assertion that an error will not occur. This
is only appropriate when the client can rule out the error by e.g. checking the
inputs it is providing to the operation.
