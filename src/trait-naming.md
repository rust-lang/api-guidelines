# Trait Naming Conventions

## Role Traits

### Naming
Nouns.

### Examples
* `std::iter::Iterator`
* `std::fmt::Debug`
* `std::fmt::Binary`
* `std::ops::Fn`
* `std::hash::Hasher`
* `futures::future::Future`
* `futures::future::Executor`

### Description
Role traits are used for specifying the role an entity plays in a system. The entities implementing it are managed by some other entities, and/or can be composited with some similiar entities into a entity with similar role.

### Rationale
The !Trait and ?Trait form doesn't apply.


## Attribute Traits

### Naming
Adjectives.

### Examples
* `std::marker::Sized`
* `std::panic::UnwindSafe`

### Description
Attribute traits are used for specifying an boolean attribute that may or may not apply to a type.

This kind of traits are especially useful as query conditions.

### Rationale
Type implementing the `Trait` form means that values of this type have an value "true" for this attribute.
Type implementing the `!Trait` form means that values of this type have an value "false" for this attribute.
`?Trait` means that both types implementing or not implementing this trait satisfy this query condition.

## Action Traits

### Naming
Verbs.

### Examples
* `std::marker::Sync`
* `std::marker::Send`
* `std::marker::Copy`
* `std::borrow::Borrow`
* `std::clone::Clone`
* `std::io::Write`
* `serde::ser::Serialize`

### Description
Action traits are used for specifying 1. values of this type are subject to `<Verb>`(name of the trait),
that is to say, they're `<Verb>`-able. 2. methods of this trait lists those actions (other than actions 
provided by core language), that can be invoked on values of this type.

This kind of traits are both useful as query conditions, and useful for providing generic entries 
for those actions, if appliable.

### Rationale
Type implementing the `Trait` form means that values of this type have an value "true" for this attribute.
Type implementing the `!Trait` form means that values of this type have an value "false" for this attribute.
`?Trait` means that both types implementing or not implementing this trait satisfy this query condition.

## Convertion Traits

### Naming
Prepositions or prepositional phrases.

### Examples
* `std::convert::Into`
* `std::convert::TryInto`
* `std::convert::AsRef`
* `std::borrow::IntoOwned`

### Description
Convertion traits are used for specifying values of this type can be transform or borrowed and represented with
mentioned types (within the trait name or parameter).

They're called explicitly to perform value transformation.

### Rationale
The !Trait and ?Trait form doesn't apply.

## Factory Traits

### Naming
Prepositions or prepositional phrases or noun(`Default`).

### Examples
* `std::default::Default`
* `std::convert::From`
* `std::convert::TryFrom`

### Description
Factory traits provide uniform ways to generate values of this type(`Self`).

They're called explicitly to perform value generation.

### Rationale
The !Trait and ?Trait form doesn't apply.
