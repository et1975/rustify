# rustify

A repo-local source pack for Rust coding guidance, using Rust project documentation for idioms
and correctness contracts, and Mainmatter's
[C to Rust Migration Book](https://mainmatter.com/c-to-rust-migration-book/course/)
as a source for mixed-language boundary engineering.

| Definition | Responsibility |
| --- | --- |
| [rust-coding](agents/rust-coding.agent.md) | Implementation workflow, local precedent, ownership and design decisions |
| [rust-validation](skills/rust-validation/SKILL.md) | Consistency and correctness gates with scoped, truthful validation evidence |
| [rust-ffi](skills/rust-ffi/SKILL.md) | Soundness and compatibility constraints for actual foreign-language boundaries |
| [Routing instructions](copilot-instructions.md) | When to delegate and invoke each definition |

The agent is for normal Rust work as well as incremental migration. FFI guidance
is conditional, not a requirement to introduce C interfaces into safe Rust.
These are Markdown definitions, not an application, compiler plugin, automated
linter, or sandbox.

## Use

The top-level `agents/` and `skills/` directories are distributable source
layout. They are not automatically active merely because this directory exists.

For a consuming Copilot repository, copy or symlink the agent into
`.github/agents/rust-coding.agent.md` and each skill directory into
`.github/skills/`, retaining its `SKILL.md`. Merge the routing instructions into
that repository's `.github/copilot-instructions.md`; preserve its existing
instructions. For personal installation, the corresponding locations are
`~/.copilot/agents/`, `~/.copilot/skills/`, and your personal instructions.
Installation is an explicit user action, not something this pack performs.

Confirm discovery with `/agent` and `/skills` in the consuming session. Select
`rust-coding` for Rust work. A delegated brief should name the goal, allowed
files, relevant existing code, repository commands, and acceptance criteria.
Keep user decisions distinct from agent recommendations. The agent needs skill
invocation access; an unavailable skill is a blocked gate, not a successful
check.

No hooks, installer, model override, binary downloads, package installs, or
global configuration changes are included. Existing Rust projects retain their
toolchain and validation policies. Optional tools such as Miri or sanitizers
are used only when applicable and already available under the project's policy.

## Design and sources

The pack separates coding workflow, validation, and boundary-specific
constraints, with consistency checks against semantically equivalent existing
code. Rust FFI constraints apply whenever an actual foreign boundary is changed.

Official-project guidance supplies Rust-specific defaults:

| Source | Applied guidance and scope |
| --- | --- |
| [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/about.html) | Recommendations authored largely by the library team, not mandates: naming, conversions, caller control, applicable traits, predictable APIs, documentation and teardown |
| [Standard conversions](https://doc.rust-lang.org/std/convert/index.html) | Infallible `From`, fallible `TryFrom`, cheap `AsRef`/`AsMut`; blanket `Into`/`TryInto` implementations |
| [The Rust Book: error handling](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html) | Expected failures use `Result`; tests and proven invariants have different panic/unwrap considerations |
| [Rust Style Guide](https://doc.rust-lang.org/style-guide/) | Recommended default formatting; preserve repository rustfmt configuration rather than imposing defaults |
| [Rust Reference: undefined behavior](https://doc.rust-lang.org/reference/behavior-considered-undefined.html) | Correctness constraints, including unsafe code; explicitly not a complete formal memory model |
| [FFI unwinding](https://doc.rust-lang.org/reference/panic.html#unwinding-across-ffi-boundaries) and [slice safety](https://doc.rust-lang.org/std/slice/fn.from_raw_parts.html) | Actual panic/ABI and pointer/slice preconditions; not interchangeable with style recommendations |

The agent turns the relevant idioms into implementation choices; validation
checks the changed surface and legitimate exceptions. The FFI skill keeps those
idioms inside the safe facade without changing a foreign contract. This does not
mandate new dependencies, blanket trait derives, new abstractions, or public API
renames. Each definition links the applicable source sections.

The course contributes incremental ABI-preserving replacement, thin adapters
around safe Rust, explicit ownership and safety contracts, C caller/differential
tests, dynamic-analysis limits, and comparable benchmarking workloads:

- [FFI validation and narrowing](https://mainmatter.com/c-to-rust-migration-book/course/02_basics/10_validate_and_narrow)
- [Replacing a module](https://mainmatter.com/c-to-rust-migration-book/course/03_intermediate/01_rewrite_a_module)
- [Unsafe-oriented lints](https://mainmatter.com/c-to-rust-migration-book/course/03_intermediate/04_clippy_lints)
- [Miri](https://mainmatter.com/c-to-rust-migration-book/course/03_intermediate/10_miri)
- [Benchmarking](https://mainmatter.com/c-to-rust-migration-book/course/03_intermediate/11_benchmarking)

The course credits Jonas Kruckenberg and Artur Jakubiec. These definitions are
an original adaptation, not a copy of the book. Sources were examined on
2026-10-22; course exercises were not run.

## Validation scope

Check frontmatter, names, routing, and local links when changing the pack. Review
the Rust rules separately for technical correctness. Exercise the guidance with
bounded scenarios covering raw-pointer validity, null/empty buffers, unchanged
C callers, allocator pairing, panic boundaries, feature/target coverage, and
unavailable tools. Include idiom cases such as fallible conversions, borrowed
versus consumed inputs, legitimate caller-buffer reuse, public examples versus
test unwraps, and custom rustfmt settings. A Markdown structure check cannot
establish Rust soundness or prove future agent behavior.

The initial unguided scenario probe already handled its safety and validation
cases correctly. No baseline failure or measured improvement is claimed; these
definitions make the requested reusable contracts explicit.
