---
name: finding-verus-resources
description: Find existing Verus examples, syntax, lemmas, documentation, and code 
---

This requires access to the Verus repository.  Look for an existing checkout (or ask).
If you don't already have a local copy of the Verus repo, identify the installed Verus 
version using `verus --version` (which will print something like
`0.2026.09.15.a1a3cbc`, where `a1a3cbc` is the corresponding commit), and then
clone the Verus repo (into a scratch directory) and checkout the matching
commit or release.  Clone one via:
```
git clone --filter=blob:none https://github.com/verus-lang/verus.git
git -C verus checkout a1a3cbc
```
Within the repo, you can find:
1. Existing function definitions and useful lemmas: `verus/source/vstd/`
2. Existing specifications for portions of the Rust standard library: `verus/source/vstd/std_specs`
2. Unit tests of Verus features: `verus/source/rust_verify_test/tests/`
3. The Verus Guide : `verus/source/docs/guide/src/`
4. Verus examples: `verus/examples/`, including snippets in `verus/examples/guide` used in the Verus guide.
5. Verus Concurrency Guide: `verus/source/docs/state_machines/src`

If for some reason you cannot clone the Verus repo,
try the online [Verus Guide](https://verus-lang.github.io/verus/guide),
and the [vstd docs](https://verus-lang.github.io/verus/verusdoc/vstd/).

## Search tips
- Try `grep -rn "proof fn lemma_" verus/source/vstd` to find existing vstd lemmas.
- Try `grep -rn "broadcast group" verus/source/vstd` to find bundles of properties that can be imported via `broadcast use`.
