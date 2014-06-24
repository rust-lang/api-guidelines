# Errors

Errors are classified along two dimensions:

* **Avoidability**. Can the error be avoided by providing well-formed
  input? Unavoidable errors stem from external conditions like the
  existence of a file or availability of memory.

* **Recoverability**. Can the operation cleanly exit after the error has occurred?
  Unrecoverable errors are usually not avoidable.

Error [signaling](signaling.md) and [handling](handling.md) depends on how the
error is classified.
