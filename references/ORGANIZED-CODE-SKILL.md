---
name: coder-organized-skill
description: "Modular architecture, well-grouped and well-named files, folders, crates/packages, and modules/namespaces; small types and functions with clear responsibilities; composition over monoliths. TRIGGER when: any .rs file is being edited, created, or reviewed, or the user asks to write/refactor/fix Rust code. DO NOT TRIGGER for: non-Rust tasks."
license: MIT.
metadata:
  short-description: Create well organized code bases, which are easy to navigate based on folders and file names alone.
---

# Organized

You are an organized coder a neat keeper of files.

# Quick References

## Separate persisted models from logic
You SHOULD separate from persisted models declarations from logic using them in different folders - by "persisted" we mean models which are saved into databases or saved to file. Logic involving these structs/enums should be kept outside of the file declaring them. This allows us to easily create a copy of a folder with models, rename all with V2 suffix and `impl From<UserV1> for UserV2 { ... }` and elsewhere `pub type User = UserV2;` and outside of the models folder, inside a sibling folder named "logic" or similar we can `impl User { fn application_logic(&self) ... }`. We can then make small adjustments to (application) logic methods when the `User` model changes over time and as long as we can map from `UserV1` to `UserV2` to `UserV3` etc we never need to duplicate logic.

### Example structure
```
crates/backend
	Cargo.toml
	src
		mod.rs
		models
			mod.rs
			v1
				mod.rs
				user.rs
		logic
			mod.rs
			user_validation.rs
```

## One file per struct/enum
You MUST create a new file for each `pub` or `pub(crate)` `struct` or `enum` (type). Smaller private helper structs or enums MIGHT be placed in the same file as other code. If the type is a persisted model, follow `#separate-persisted-models-from-logic`, else put methods inside the same file as where the type is declared.

## Small code units

### Small functions and methods
For new projects, create a `.clippy.toml` file and set `too-many-lines-threshold = 20`.

You should ONLY bypass `too-many-lines-threshold` (or similar lints/checks, e.g. manually written in build.rs enforcing max Lines of (non-test) Code in a file) if it the body matches on a big enum - but try to not make enums with more than ~10 variants in first place if you do not have a very good reason for it, try to nest/group the variants instead. Examples where we might want a completely flat enum and thus ALLOW bypassing `too-many-lines-threshold` is a Localization/Internationalization enum, or type discriminators / metatype e.g.:

#### Examples where it is OK to bypass max LOC check
```rust
/// Example of enum which we prefer being "flat" (more than ~10 variants)
pub enum ValueKind {
    Bool,
    I8,
    I16,
    I32,
    I64,
    I128,
    U8,
    U16,
    U32,
    U64,
    U128,
    String,
    Enum,
    Array,
    Tuple,
    Map,
    Reference,
    Own,
    Decimal,
    PreciseDecimal,
    NonFungibleLocalId,
}
```

### Small files
You write small files, typically less than 500 LOC.