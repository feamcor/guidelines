# Rust Coding Guidelines

This document defines rules and decision criteria for an automated or human coding agent writing **production Rust code**. It prioritizes correctness, maintainability, and explicitness over convenience or brevity.  

## 1. Core Principles (Non-Negotiable)

### 1.1 Make Invalid States Unrepresentable
- Prefer the type system over runtime checks.
- Encode constraints in types.
- Use distinct types for semantically different values.

### 1.2 Ownership Is a Design Tool
- Own data by default.
- Borrow only when performance or semantics require it.
- Avoid explicit lifetimes unless unavoidable.

### 1.3 Fail Early, Fail Clearly
- Errors must be typed and contextual.
- Panics are allowed **only** for programmer errors.
- No silent failures.

### 1.4 Small, Focused Modules
- One responsibility per module.
- Minimal public API.
- Private internals by default.

## 2. Project Structure & Modularization

### 2.1 Crate Layout

```text
src/
  lib.rs          # Public API surface
  main.rs         # Wiring only (if binary)
  error.rs        # Error types
  types/          # Domain types (newtypes, value objects)
  services/       # Business logic
  adapters/       # IO: DB, HTTP, FS, external APIs
  utils/          # Pure helpers (no side effects)
```

### 2.2 Visibility Rules
- Default visibility: `pub(crate)`
- Use pub only for intentional cross-crate APIs.
- Avoid pub struct with public fields.
- Expose behavior, not data.

## 3. Types & Newtypes (Mandatory Where Applicable)

### 3.1 When to Use Newtypes
Use a newtype when:
- A primitive has semantic meaning.
- Units matter (IDs, durations, sizes).
- Values must not be mixed.

### 3.2 Constructors & Validation
- No public fields.
- No unchecked constructors.
- Prefer `TryFrom`, `FromStr`, or smart constructors.

## 4. Error Handling Strategy

### 4.1 Error Crates
- For Binary/Application, use `anyhow`.
- For Library/Domain, use `thiserror`.
- For Public API, use typed errors only.

### 4.2 Defining Errors (Library Code)
- Errors must be domain-specific.
- No String-only error variants.
- Include relevant context data.

### 4.3 Propagating Errors (Binary Code)
- Attach context at IO boundaries.
- Never use `unwrap()` or `expect()` outside tests.

## 5. Result, Option & Control Flow

### 5.1 Option vs Result
- When absence is expected, use `Option`.
- When failure is exceptional, use `Result`.
- Upon validation failure, use `Result`.
- Never encode errors as `None`.

### 5.2 Control Flow Rules
- Prefer `let/else`.
- Prefer early returns.
- Avoid deep nesting.
- No `else` after `return`.

## 6. Traits & Abstractions

### 6.1 Trait Design
- Traits represent capabilities, not data.
- Avoid “fat” traits.
- Prefer associated types when clearer.

### 6.2 Dynamic Dispatch (dyn Trait)
Use only when:
- Multiple implementations are selected at runtime.
- Plugin-style extensibility is required.
- Otherwise, use generics.

## 7. Ownership, Borrowing & Cloning

### 7.1 Cloning Rules
- Cloning must be explicit and intentional.
- `Arc`/`Rc` clones are acceptable.
- Large `String` or `Vec` clones require justification.

### 7.2 Input vs Storage Types
- Use `&str` for function inputs.
- Use `String` for owned storage.

## 8. Async & Concurrency

### 8.1 Async Boundaries
- Async only at system boundaries (web services, databases, files).
- Domain logic must remain synchronous.
- Avoid async_trait unless unavoidable.

### 8.2 Concurrency Primitives
- For Shared ownership, use `Arc<T>`.
- For Shared mutable state, use `Mutex<T>`.
- For Read-heavy contention, use `RwLock<T>`.
- For Message passing, use `tokio::sync::mpsc`.
- Avoid `std::sync` primitives in async contexts.

## 9. Testing Standards

### 9.1 Unit Tests
- One module, one test module.
- Test behavior, not implementation.
- No mocks for pure logic.

### 9.2 Integration Tests
- Prefer real implementations.
- Isolate external resources.
- No shared mutable state between tests.

## 10. Logging & Observability
- Use `tracing`.
- Never use `println!`.
- Prefer structured logging.

## 11. Formatting, Lints & Tooling
Mandatory:
- `rustfmt` (no overrides)
- `clippy --all-targets --all-features`

Rules:
- Clippy warnings must be fixed.
- Allowances require explicit justification.

## 12. Disallowed Anti-Patterns
- `unwrap()` / `expect()` outside tests.
- Public struct fields.
- Stringly-typed identifiers.
- Errors as raw strings.
- Business logic in `main.rs`.
- Overusing lifetimes to avoid cloning.
- Unbounded “utils” modules.

## 13. Agent Decision Checklist
Before writing code, verify:
- Should this be a newtype?
- Is this error domain-specific?
- Is this module responsible for IO or business logic?
- Can the type system enforce this invariant?
- Is cloning justified?
- Does this need to be public?

When in doubt, default to:
- Stricter typing.
- Narrower visibility.
- Clearer errors.
