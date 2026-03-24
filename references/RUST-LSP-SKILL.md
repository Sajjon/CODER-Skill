---
name: RUST-LSP-Skill
description: >
  LSP-first Rust development workflow for Claude Code. Use this skill whenever
  working on Rust (.rs) files, Cargo projects, or any Rust codebase. Covers
  navigation, diagnostics, refactoring, formatting, and pre-write inspection.
  Uses rust-analyzer LSP for navigation/inspection, and cargo for diagnostics
  and formatting (LSP diagnostics, rename, codeAction, formatting not yet
  supported in Claude Code).
---

<!--
Informed by https://karanbansal.in/blog/claude-code-lsp/
and https://github.com/anthropics/claude-code/issues/15302
-->

# RUST-LSP-Skill — Rust LSP-First Workflow

The rust-analyzer LSP plugin is active for all `.rs` files. Use the commands
below — do not improvise with raw grep, sed, or manual multi-file edits.

**What LSP supports in Claude Code today:**
`goToDefinition`, `goToImplementation`, `findReferences`, `hover`,
`documentSymbol`, `workspaceSymbol`, `incomingCalls`, `outgoingCalls`

**Not yet supported in Claude Code LSP** (use cargo fallbacks instead):
`getDiagnostics` → use `cargo check`
`rename` → use `findReferences` + `str_replace` per file
`codeAction` → write fixes manually guided by cargo output
`formatting` → use `cargo fmt`

---

## Commands Reference

| Command | What it does | LSP or cargo? |
|---|---|---|
| `/rust-check` | errors + warnings | `cargo check` |
| `/rust-rename` | find all sites then rename per file | LSP `findReferences` + `str_replace` |
| `/rust-fix` | apply fixes for errors/warnings | manual, guided by `cargo check` output |
| `/rust-map` | list symbols in file or workspace | LSP `documentSymbol` + `workspaceSymbol` |
| `/rust-refs` | all usages + call hierarchy | LSP `findReferences` + `incomingCalls`/`outgoingCalls` |
| `/rust-inspect` | signature, docs, implementation | LSP `hover` + `goToDefinition` |
| `/rust-format` | format files | `cargo fmt` |
| `/rust-refactor` | full safe refactor workflow | LSP nav + `cargo check` + `str_replace` |

---

## Core Rules (non-negotiable)

**Navigation — use LSP, never grep**
- Use LSP over grep for all code navigation
- Use `/rust-map` and `/rust-refs` to explore structure — never read whole files

**Diagnostics — use cargo check, never cargo build**
- Run `/rust-check` (`cargo check`) after EVERY `.rs` edit
- Fix all errors and warnings before the next change
- For unimplemented forward references use:
  `todo!("Claude: implement this before end of session")`

**Refactoring — findReferences first, str_replace per file, never sed**
- Always use `/rust-rename` (findReferences + str_replace) — never raw `sed`
- Always use `/rust-fix` for known error patterns — never skip errors
- Always run `/rust-refactor` before a large refactor

**Writing code — inspect before writing**
- Always run `/rust-inspect` before calling an unfamiliar type or function
- Always run `/rust-map` on the target file before deciding where to add code

**Formatting — cargo fmt, never manual**
- Never manually reformat Rust code — always use `/rust-format` (`cargo fmt`)

---

## Workflow Cheatsheet

### Starting a coding task
1. `/rust-map <target_file>` — understand what's already there
2. `/rust-inspect <type_or_fn>` — understand what you're working with
3. Write code
4. `/rust-check` — fix errors immediately
5. `/rust-format` — clean up

### Renaming anything
1. `/rust-refs <symbol>` — see blast radius
2. `/rust-rename <old> <new>` — findReferences + str_replace per file
3. `/rust-check` — verify clean

### Before a large refactor
→ `/rust-refactor <description>` — handles the full map → plan → execute → verify sequence

### Fixing errors after an edit
→ `/rust-fix` — reads cargo check output and applies corrections via str_replace

---

## What grep is still good for

Use grep/ripgrep only for:
- Searching inside string literals or comments
- Finding TODO/FIXME/HACK markers
- Searching `Cargo.toml` or non-Rust config files
- Pattern matching across non-`.rs` files
