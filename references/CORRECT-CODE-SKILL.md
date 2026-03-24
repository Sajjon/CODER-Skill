---
name: coder-correct-skill
description: "Write code with strong type modeling, validation, impossible states made impossible, meaningful invariants, and expressive newtypes. TRIGGER when: any .rs file is being edited, created, or reviewed, or the user asks to write/refactor/fix Rust code. DO NOT TRIGGER for: non-Rust tasks."
license: MIT.
metadata:
  short-description: Write correct code, by impossible states made impossible in models.
---

# Correct
Model types according to the semantic of the intrinsic constraints of the entity being modelled - a person cannot have negative age, thus for an `age` field do not use a signed integer, furthermore it is not reasonable a person is older than 255 years, so use `u8` and not something larger. A `User` struct's email cannot be "foo", since it is easy to validate email address do that.

## Strong type modelling
Instead of using `age: u8` we should wrap the primitive in its own type, where we can further validate the value, add string formatting conveniences, and add methods.

AVOID composing models of primitive values directly when they have a clear semantic meaning
```rust
/// AVOID this
struct Person {
  /// u8 is not as good as a newtype
  age: u8
}
```

PREFER modelling age like this:
```rust
use bon::bon;

type RawValue = u8;
pub struct Age(RawValue);
#[bon]
impl Age {
  const MAX_AGE: RawValue = 150;

  #[builder]
  pub fn new(value: u8) -> Result<Self, Error> {
    if value > MAX_AGE { 
      return Err(Error::InvalidAge { got: value, expected_at_most: MAX_AGE }) 
    }
    Ok(Self(value))
  }
}
```

Even more important to avoid using `String`s everywhere, strongly prefer to wrap `String`s in newtypes, or 

## Impossible states made impossible
Try to think of systems as finite state machines (FSM), were states are accurately represented and disallow non-sensical state. Enums / Tagged Unions / ADT with associated values are a great tool for this - languages which lacks proper support for it, try to emulate them (somewhat doable in Go, Typescript).

Imagine we want to model a lockable door. A bad approach would be:
```rust
/// Do NOT do this
struct LockableDoor {
  is_locked: bool,
  is_open: bool,
}
```

Instead we should model it as an enum, as a simple Finite State Machine (FSM):
```rust
enum LockableDoor {
  /// Open (thus not locked)
  Open,
  /// Closed, but not locked
  ClosedUnlocked,
  /// Closed and locked
  ClosedLocked,
}
```

And we can also disallow nonsensical state.

## DRY
DO **NOT** REPEAT YOURSELF: You declare constants and reuse them. Everywhere, always, like we did with `Age` above:

PREFER
```rust
impl Age {
  const MAX_AGE: RawValue = 150;
  pub fn new(value: u8) -> Result<Self, Error> {
    if value > MAX_AGE { 
      return Err(Error::InvalidAge { got: value, expected_at_most: MAX_AGE }) 
    }
    Ok(Self(value))
  }
}
```

AVOID
```rust
impl Age {
  pub fn new(value: u8) -> Result<Self, Error> {
    if value > 150 { // DO NOT do this, have to declare `150` again below
      return Err(Error::InvalidAge { got: value, expected_at_most: 150 }) // 150 should be a constant!
    }
    Ok(Self(value))
  }
}
```