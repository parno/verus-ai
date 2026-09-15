---
name: running-verus
description: Run Verus or cargo verus to check Rust code. Use whenever verifying code, re-checking after edits, or narrowing verification to one crate, module or function.
---

These instructions assume you already have the `verus` binary on your PATH.
If it isn't, ask the user.

For verification projects that only verify a single crate, you can run Verus
directly on the crate root:
```
verus src/main.rs
```
or
```
verus --crate-type=lib src/lib.rs
```
For projects that involve multiple verified crates, use `cargo-verus` (which should 
be installed in the same directory as the `verus` binary):
```
cargo verus verify
```
Run `cargo verus -h` for more options.

Note: `cargo verus` only verifies crates that have
```
[package.metadata.verus]
verify = true
``` 
in their `Cargo.toml`

## Interpretting the output

A successful verification outcome should display "verification results:: N verified, 0 errors" and have a 0 exit code.  `cargo verus` runs Verus once perf verification-enabled crate, so check every such summary line.

Run with `--output-json` for machine-readable results.

When a proof fails, use the Verus argument `--expand-errors` for more detailed error reporting. 

By default, Verus only reports at most 2 errors per function.  To see more,
use the Verus argument `--multiple-errors N` for some number N > 2.

## Faster verification checking

In a large proof development, you can restrict the scope of
verification in order to get results faster:
1. An individual crate and its dependencies: `cargo verus verify -p crate-name`
2. An individual crate without its dependencies: `cargo verus focus -p crate-name`
3. An individual module and its submodules `--verify-module mod-name`.  It accepts paths, like `foo::bar`.
4. An individual module without its submodules `--verify-only-module mod-name`.  It accepts paths, like `foo::bar`.
5. An individual function `--verify-module mod-name --verify-function function-name`
   (or use `--verify-root` if the function is not within any other module).  It matches
   a unique substring or supports `*foo*` wildcards.

When passing Verus arguments using `cargo`, put them after a `--` argument, e.g.,
`cargo verus focus -p crate-name -- --verify-module module-name`.

Note: `--verify-root`, `--verify-module`, `--verify-only-module`, and
`--verify-function` **only** work with `verus` or `cargo verus focus`.  They
**do not** work with `cargo verus verify`.

**IMPORTANT**: You cannot consider verification to have fully succeeded until
you run **without** these restrictions.


