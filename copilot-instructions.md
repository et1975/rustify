# Rust Instructions

These instructions apply to Rust implementation work, including `.rs` files and
Rust-related `Cargo.toml` changes. They do not require Rust tooling for
documentation-only changes.

- Delegate Rust implementation to the `rust-coding` agent when it is available.
  Give it the user's objective and decisions with attribution, the allowed files,
  relevant source paths, existing build/test commands, and observable acceptance
  criteria. Identify agent-proposed designs as proposals, not user decisions.
- For a small change, keep that brief short. Do not create another wrapper,
  module, error type, or abstraction before examining an existing equivalent
  with the same semantic role and invariants.
- Require `rust-validation` after Rust edits and for Rust reviews. Assign one
  validation owner per attempt; the caller checks the returned evidence rather
  than repeating an identical pass. Revalidate after follow-up edits or when
  evidence does not cover the requested behavior.
- Invoke `rust-ffi` when changing an actual foreign-language boundary: C ABI
  exports/imports, foreign pointers, callbacks, allocation ownership, or a
  C-to-Rust migration. Ordinary safe Rust does not need an FFI overlay.
- Preserve repository toolchain, MSRV, edition, feature, target, and dependency
  conventions. Do not silently install toolchains, components, tools, or
  dependencies. Missing prerequisites are explicit validation limitations.
- If an agent or skill is unavailable, report it. Do not claim it ran, silently
  install it, or claim a completed validation gate. Independently useful work
  may continue with the missing gate reported as blocked.

Use repository-native commands and configuration. Formatting, linting, unit
tests, C integration tests, and dynamic analysis provide different evidence;
none substitutes for all the others.
