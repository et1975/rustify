---
name: rust-ffi
description: Use when working with extern C boundaries, raw foreign pointers, ownership crossing an ABI, callbacks, generated bindings, or C-to-Rust migration.
---

# Rust FFI

## Announcement

> Using rust-ffi to preserve the foreign contract and audit the boundary's safety obligations.

## Invariant

Keep the foreign adapter thin and the core safe. Every boundary operation must preserve its actual ABI and ownership contract, with explicit obligations for callers and implementers. Generated bindings describe declarations; they do not establish safe usage. Ordinary safe Rust code has no blanket C ABI requirement.

This reference defines review obligations, not a soundness proof or an empirically demonstrated improvement over unguided work.

## Checklist

### 1. Establish the actual contract

Record the authoritative header/API version, exported symbols, calling convention, return/error codes, platform targets, layout/alignment, and ownership rules. Preserve these during migration unless a contract change is explicitly authorized. Verify platform-dependent primitive widths; C `long`, `char`, and Rust pointer-sized integers are not interchangeable by assumption.

Use ABI-appropriate types and explicit conversion at the adapter. `#[repr(C)]` controls layout; it does **not** make arbitrary foreign bit patterns valid Rust enums, `bool`s, references, or nested values. Accept an appropriate integer representation and validate it before constructing a restricted enum. Do not expose Rust `Vec` or `String` layout as a stable C interface.

Keep Rust API idioms behind that adapter: meaningful `Result` errors, methods on owned handles, and tuple/struct returns where appropriate. Preserve foreign symbols, out-parameters, buffer-reuse contracts, and status codes even when they differ from Rust naming or return conventions. Standard safe conversions (`From`/`TryFrom`) cannot impose uncheckable pointer-validity preconditions: such construction needs an unsafe API with documented `# Safety` obligations or an owner-based abstraction that actually enforces them.

### 2. Prove pointer and length obligations

- Document whether NULL is accepted, the allocation owner, accessible initialized range, alignment, mutability, aliasing/exclusivity, and required lifetime. Non-nullness alone proves none of validity, liveness, or legal access. Invalid foreign pointers cannot generally be runtime-validated.
- For pointer-plus-length input permitting `(NULL, 0)`, branch to an empty representation **before** calling `from_raw_parts`. That operation still requires a non-null aligned pointer for zero elements. Never construct a reference or slice first and validate it afterward.
- Check integer conversions and byte-size arithmetic for overflow. A slice's range must lie within one allocation and its total size must not exceed `isize::MAX`; also respect element alignment and initialized-value requirements. Check any stricter API limit.
- Tie borrowed lifetimes to the actual owner or an enforceable call-scoped contract; do not invent an unconstrained lifetime from a raw pointer. A safe facade must prevent callers from violating these obligations.
- Distinguish output capacity from initialized length. For `MaybeUninit` buffers, establish exactly which elements the foreign operation initialized on every return path before reading them, setting a length, or calling `assume_init`. Preserve failure cleanup.

### 3. Pair ownership and destruction

Write down which library allocated each resource and the exact matching destructor, including who calls it and when. Do not mix C allocation/free with Rust allocation/destruction. `Box::from_raw` requires the corresponding compatible `Box` allocation and a single ownership recovery; an arbitrary pointer or a second recovery is not sufficient.

For strings, distinguish borrowed NUL-terminated `CStr`, owned `CString`, and length-delimited bytes. Prove termination and readable bounds before constructing `CStr`; it does not imply UTF-8. Use checked decoding or the contract's explicit byte policy. `CString::into_raw` ownership must return through its matching recovery contract, not foreign `free`; account for restrictions on foreign mutation before reclaiming it.

Prefer explicit handles and matching release functions over exposing implementation layout. Account for retained pointers, double release, partial initialization, and destruction after the library or owner has gone away.

For an owned Rust facade, pair resource ownership with appropriate RAII cleanup rather than requiring every Rust caller to remember a release call. Keep an explicit fallible close/finalize operation when callers need teardown errors; do not panic from `Drop` to report ordinary cleanup failure. Document fallback cleanup and retained-pointer rules.

### 4. Audit callbacks, threads, and unwinding

Specify callback registration/unregistration, context-pointer ownership, in-flight calls, lifetime, thread affinity, concurrency, and reentrancy. A callback may outlive the registration call. Justify any unsafe `Send`/`Sync` implementation using the foreign library's documented guarantees, not the presence of a lock or a raw pointer alone.

Panics must not unwind through a non-unwinding C ABI. Follow the actual boundary strategy: return codes, error objects, containment, or documented abort behaviour. `catch_unwind` is not pointer validation, cannot catch aborting panics, and requires an appropriate unwind-safety analysis. It does not make undefined behaviour recoverable. Where C++ exceptions or unwind-capable ABIs are involved, verify the explicit cross-language exception agreement and toolchain support rather than assuming either direction is safe.

Do not rely on `catch_unwind` as a C++ exception catcher: the Reference permits abort or an opaque error when catching a foreign unwind, with unspecified choice. Distinguish this from containing supported Rust panics. Document relevant error/panic behavior and unsafe caller obligations separately.

### 5. Validate compatibility and performance claims

Use the repository's smallest relevant Rust tests plus a C caller/harness for exposed contract changes. Differential tests against the existing implementation should compare valid inputs, documented invalid inputs, return codes, ownership, and observable behaviour. Never deliberately dereference an invalid pointer as an ordinary negative test.

Reuse the exact approved, preprovisioned pinned toolchain. Before invoking Cargo or rustup proxies, verify installed toolchain/components without provisioning: proxies can implicitly download missing pins, and Cargo can fetch dependencies. Honor repository dependency-fetch policy, using offline/no-fetch facilities where needed. Do not automatically download/install prerequisites without explicit approval. Missing unauthorized prerequisites make checks **blocked**, not failed evidence of the code itself.

Miri can examine supported Rust-only adapter/core paths; it generally cannot execute native C. Claim a special supported mode only with documented setup and limitations. Use sanitizers or native harness checks when relevant, supported, and already provisioned; neither proves complete soundness.

Benchmark equivalent work: same inputs, validation, allocations, ownership transfers, optimization, and observable outputs. Prevent dead-code elimination, repeat measurements, and report comparable medians and variability rather than one convenient timing.

## Common mistakes

| Mistake | Correction |
|---|---|
| “bindgen generated it, so it is safe” | Audit the usage contract and safe abstraction separately. |
| “NULL plus zero is a valid slice” | Handle that contract before slice construction. |
| “repr(C) validates an enum” | Validate integer input before constructing restricted values. |
| “catch_unwind catches bad pointers” | Prove memory obligations independently of panic handling. |

## Report shape

- **Contract:** header/symbol/platform references and preserved behaviour.
- **Safety:** pointer, lifetime, allocation/destructor, callback, and unwind obligations.
- **Evidence:** exact executed commands/results, C/differential coverage, benchmark conditions.
- **Limits:** untested configurations, blocked tools, caller obligations, unresolved risks.

## Sources

Official correctness contracts: [Reference undefined behavior](https://doc.rust-lang.org/reference/behavior-considered-undefined.html), [FFI unwinding](https://doc.rust-lang.org/reference/panic.html#unwinding-across-ffi-boundaries), and [slice::from_raw_parts safety](https://doc.rust-lang.org/std/slice/fn.from_raw_parts.html). Follow the safety documentation of each actual operation; the Reference's UB list and aliasing description are explicitly not an exhaustive formal model.

Official-project design recommendations, not ABI-changing mandates: [API Guidelines scope](https://rust-lang.github.io/api-guidelines/about.html), [predictable Rust APIs](https://rust-lang.github.io/api-guidelines/predictability.html), [documentation](https://rust-lang.github.io/api-guidelines/documentation.html), [teardown](https://rust-lang.github.io/api-guidelines/dependability.html), and [safe conversion traits](https://doc.rust-lang.org/std/convert/index.html).

Supplementary migration teaching from Mainmatter: [validate and narrow](https://mainmatter.com/c-to-rust-migration-book/course/02_basics/10_validate_and_narrow), [rewrite a module](https://mainmatter.com/c-to-rust-migration-book/course/03_intermediate/01_rewrite_a_module), [Miri](https://mainmatter.com/c-to-rust-migration-book/course/03_intermediate/10_miri), [benchmarking](https://mainmatter.com/c-to-rust-migration-book/course/03_intermediate/11_benchmarking). Reviewed 2026-10-22; not a claim of completed exercises.
