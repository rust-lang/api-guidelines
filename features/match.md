% Pattern matching

### Dereference `match` targets when possible. [RFC]

Prefer

~~~~
match *foo {
    X(...) => ...
    Y(...) => ...
}
~~~~

over

~~~~
match foo {
    box X(...) => ...
    box Y(...) => ...
}
~~~~

<!-- ### Clearly indicate important scopes. **[RFC]** -->

<!-- If it is important that the destructor for a value be executed at a specific -->
<!-- time, clearly bind that value using a standalone `let` -->
