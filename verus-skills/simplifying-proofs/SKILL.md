---
name: simplifying-proofs
description: After developing Verus proofs that successfully verify, use this skill when asked to clean up, simplify, or minimize the proofs.
---

Only use once the proof verifies; run Verus with the `running-verus` skill first to confirm.

# Steps to Simplify Proofs

After each step below, use the `running-verus` skill to check that the proofs still succeed.

1. Is there any dead spec/proof code that can be removed?  Repeat this process until nothing more can be removed.
2. Are there more assertions than necessary?  If so, remove them.  Typically you only need assertions to trigger quantifiers, to invoke extensionality (e.g., for sequences, sets, and maps), to invoke a specialized solver mode (e.g., for the bit-vector solver), or when structuring a proof using `assert(e) by { ... }` or `assert forall |x| ... implies ... by {}`.
3. You typically only need `reveal(function_name)` when `function_name` is annotated with `#[verifier::opaque]`.  If you reveal it everywhere you use the function, then consider removing the annotation.
4. Consider whether function contracts can be simplified.  Complex spec expressions that are used more than once should be factored out into a named spec function that can be reused.  Even if used only once, factoring into a spec function can more clearly convey the semantic intent of the expression.  This is allowed for public APIs, as long as the spec remains semantically identical.  For example, consider converting:
    ```
    fn binary_search(v: &Vec<u64>, k: u64) -> (r: Option<usize>)
        requires 
            forall|i: int, j: int| 0 <= i <= j < v.len() ==> v[i] <= v[j]
        ensures ...
    {}
    ```
    into
    ```
    spec fn sorted(v:Seq<u64>) -> bool {
        forall|i: int, j: int| 0 <= i <= j < v.len() ==> v[i] <= v[j]
    }
    
    fn binary_search(v: &Vec<u64>, k: u64) -> (r: Option<usize>)
        requires 
            sorted(v@)
        ensures ...
    {}
    ```
5. Are there multiple lemmas proving special cases of the same general idea?  Consolidate them into a general lemma reduce the overall proof code.

# IMPORTANT NOTES
- Do not change the executable code.  Changes to proof blocks, `proof fn`, ghost code, and loop invariants are all fair game.
- Do not change the public API specifications.
- Ensure that the proofs still verify after your changes.
