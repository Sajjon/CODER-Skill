---
name: coder-rigorous-skill
description: "Extremely high code standards, linting, typo checks, supply-chain care, 90%+ test coverage, CI/CD. TRIGGER when: any .rs file is being edited, created, or reviewed, or the user asks to write/refactor/fix Rust code. DO NOT TRIGGER for: non-Rust tasks."
license: MIT.
metadata:
  short-description: Write code with extremely high standards
---


# Rigourus

## Lint
Enable pedantic lints and treat

## Avoid abbreviation - claity is more important than brevity
ONLY abbreviate inside closures in long functional chains:
```rust
response.users
  .filter(|u| u.is_logged_in())
  .filter(|u| u.is_swedish())
  .map(Swedish)

```


# Dependency management
**DO** pin rust crates to specific semver, e.g. `serde = { version = "1.0.228" }`.