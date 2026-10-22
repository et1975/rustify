---
name: rust-validation
description: Use when Rust .rs files, Cargo.toml manifests, cfg settings, or build scripts change, or when reviewing Rust code.
---

# Rust Validation

## Announcement

> Using rust-validation to check local consistency, correctness, and the evidence covering this change.

## Invariant

A Rust change is validated only to the extent supported by actual checks of the relevant code and configurations. Preserve established contracts; never substitute familiar style, a clean lint run, or an unexecuted command for evidence.

This is a self-contained reference and validation gate, not proof of soundness or a claim that guidance improves agent performance.

## Checklist

### 1. Consistency with neighbours — first

- Find pre-existing code serving an equivalent **semantic role**: public value, internal identifier, borrowed view, owned resource, fallible operation, or boundary adapter. Compare its invariants and callers, not its field count or superficial shape. Newly authored siblings are not independent precedent.
- Preserve the repository's edition, minimum supported Rust version (MSRV), public API, dependency policy, feature conventions, and error vocabulary. Check manifests, toolchain/configuration files, documented commands, and relevant CI jobs.
- Mirror equivalent representation and error patterns where they preserve the required semantics. Do not mandate a wrapper for every single-field value, replace every error with a new abstraction, or ban all `unwrap` calls: tests and documented established invariants have different roles from recoverable production failures.
- Correctness outranks precedent. Identify an unsound or incompatible pattern rather than copying it. Report necessary deviations and their reason; avoid unrelated cleanup.

The Rust API Guidelines are contextual recommendations, not language rules. The Style Guide defines recommended defaults, not a ban on custom rustfmt settings. Apply idiom checks to the changed Rust-facing surface; do not rename existing APIs, change an ABI, add dependencies, or override deliberate compatible local style simply to match a checklist.

### 2. Review the changed contracts

- Trace ownership, borrowing, lifetimes, aliasing, drop order, and resource cleanup on success and error paths. Ensure error propagation preserves meaningful context without unintended panics or swallowed failures.
- For asynchronous code, check cancellation at suspension points, partial effects, blocking work on executor threads, lock scope, and thread-safety requirements. A type compiling with `Send` or `Sync` does not settle the application's concurrency contract.
- For public APIs, inspect compatibility, useful rustdoc examples, doctests, and applicable `Errors`/`Panics`/unsafe `Safety` sections. Prefer `?` in fallible public examples; do not extrapolate this to banning `unwrap` in tests or `expect` for an explained, proven invariant. Exercise new behaviour and regressions at the smallest useful unit or integration boundary.
- Keep `unsafe` narrowly scoped with a specific safety justification for each obligation. Inspect caller requirements as well as implementation. Actual foreign ABI, pointer, allocation, or callback boundaries require boundary-specific review; the separately discoverable `rust-ffi` skill covers those cases, but ordinary safe Rust does not acquire C ABI constraints.

For changed Rust APIs, review these applicable idioms and record material exceptions:

- **Naming and conversions:** Rust casing; ordinary `field`/`field_mut` getters versus checked collection `get`; `as_` cheap borrowed views, `to_` potentially doing work, `into_` consumption. Check `iter`/`iter_mut`/`into_iter` ownership modes. Prefer `From`/`TryFrom` over direct `Into`/`TryInto` impls on supported MSRVs; fallible validation must not become a panicking `From`.
- **Caller control:** borrow when possible, consume when required; no hidden clone solely because an API borrowed unnecessarily. Use text/slice views or minimal useful iterator/I/O bounds rather than imposing an owned container. Multiple newly produced values normally belong in returned tuples/structs; intentional caller-buffer reuse remains valid.
- **Traits and errors:** check appropriate common traits, consistent `new()`/`Default`, and meaningful public errors with `Debug`/`Display` and an available `Error` trait compatible with `std`/`no_std` and MSRV. Preserve causes and expected-error propagation. Do not blanket-derive resource traits, add serialization dependencies, or use arbitrary newtype `Deref` as inheritance.
- **Teardown:** do not panic to report ordinary teardown failure from `Drop`. When callers need fallible or potentially blocking completion, offer an explicit close/finalize operation and preserve the repository's fallback cleanup/error-reporting policy.

### 3. Select checks that cover this change

Use repository commands and configured policies before generic recipes. Choose the smallest package/test selection covering changed behaviour, expanding when shared crates, build scripts, configuration, or initial results require it. Include relevant default/no-default-feature combinations, supported targets, and CI configurations; explain omitted configurations.

These are **illustrative**, not executed commands. Replace `crate_name` and adapt flags to the actual repository:

```text
cargo fmt --all -- --check
cargo clippy -p crate_name --all-targets --offline
cargo test -p crate_name --offline
cargo test -p crate_name --doc --offline
```

Use the exact approved, preprovisioned pinned toolchain, including when invoking these examples. Before invoking Cargo or a rustup tool proxy, verify that the required toolchain and components are already installed using non-provisioning inventory checks. A proxy can implicitly download a missing pinned toolchain; omitting `rustup install` does not prevent that. Cargo can also fetch dependencies: honor the repository's dependency-fetch policy, using offline operation where needed. Do not automatically install or download toolchains, components, packages, or helper executables without explicit approval. Normal Cargo checks using available approved prerequisites remain appropriate. If required prerequisites are absent and fetching is not authorized, report that check as **blocked**, not passed.

Apply the repository's lint policy. Add `-- -D warnings` only when that policy requires it. `--all-targets` within a selected package is not coverage of every platform target. Do not indiscriminately use `--all-features`: some features are mutually exclusive. Workspace-wide checks are warranted by actual scope, not mandatory by habit.

### 4. Interpret evidence honestly

- Formatting verifies formatting. `cargo check` verifies compilation/type checking for its selected configuration, not runtime behaviour. Clippy diagnostics are useful, not a soundness proof.
- Run relevant tests, including public examples/doctests when affected. A cross-target compile is not execution on that target.
- For relevant unsafe Rust, use already provisioned Miri when supported and useful. State its exercised Rust-only subset and limitations. Native C execution is generally unsupported; use an exceptional supported mode only with its documented setup and scope identified.
- Use existing sanitizer or integration facilities when relevant and available. Passing dynamic checks covers exercised paths, not every possible input or interleaving.
- Assign one validation owner per attempt. A caller verifies that owner's commands, results, and coverage evidence; do not repeat full passes without a changed artifact, missing coverage, or another stated reason. Revalidate affected checks after fixes.

## Common mistakes

| Mistake | Correction |
|---|---|
| Copying a similarly shaped new type | Compare pre-existing equivalent roles and invariants. |
| Applying every API recommendation as a hard rule | Review changed contracts and scoped defaults; preserve legitimate exceptions. |
| Treating Clippy or compilation as runtime proof | Identify which tests actually executed the behaviour. |
| Turning on every feature to claim completeness | Select supported combinations, including exclusions. |
| Installing a missing checker silently | Mark it blocked and describe the missing prerequisite. |

## Report shape

1. **Scope and consistency:** changed contracts, equivalent precedent, applicable idiom checks and deviations.
2. **Checks:** exact commands, working directory, toolchain, configurations, observed results.
3. **Coverage:** runtime/doctest/platform/unsafe coverage and exclusions.
4. **Unavailable or unresolved:** blocked checks, remaining risks, and owner.

Never report suggested commands as executed or an unavailable check as success.

## Sources

Official-project guidance: [API Guidelines scope](https://rust-lang.github.io/api-guidelines/about.html), [naming](https://rust-lang.github.io/api-guidelines/naming.html), [interoperability](https://rust-lang.github.io/api-guidelines/interoperability.html), [flexibility](https://rust-lang.github.io/api-guidelines/flexibility.html), [predictability](https://rust-lang.github.io/api-guidelines/predictability.html), [documentation](https://rust-lang.github.io/api-guidelines/documentation.html), [dependability](https://rust-lang.github.io/api-guidelines/dependability.html), [standard conversions](https://doc.rust-lang.org/std/convert/index.html), [Book error-handling guidance](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html), and [Style Guide](https://doc.rust-lang.org/style-guide/).

Correctness constraints: [Reference undefined behavior](https://doc.rust-lang.org/reference/behavior-considered-undefined.html) and the standard-library safety contracts of the operations used. The Reference explicitly does not provide an exhaustive, fully formal unsafe model.

Supplementary migration teaching: Mainmatter's [Clippy](https://mainmatter.com/c-to-rust-migration-book/course/03_intermediate/04_clippy_lints) and [Miri](https://mainmatter.com/c-to-rust-migration-book/course/03_intermediate/10_miri) chapters. Sources reviewed 2026-10-22; this does not imply exercise completion.
