---
name: debugging-verus-proofs
description: Diagnose and repair failing Verus proofs. Use when Verus reports verification errors like "assertion failed", "postcondition not satisfied", "invariant not satisfied", or "rlimit exceeded".
---

Syntax, type checking, mode, ghost/tracked lifetime errors **are not** proof failures.  Fix them first before trying these techniques.

Use the narrowing flags from the `running-verus` SKILL to focus on the failing proof, then do a full verification run.

## Understand the failure

- Use the `--expand-errors` flag, which can often help pinpoint which part of an obligation is failing.
- Think about **why** the failing property should hold.  What facts are needed?
- Use assertions of intermediate properties to identify where your expectations diverge from what the solver believes is true.  See the Verus Guide's section on "Using assert and assume to develop proofs" (use the finding-verus-resources SKILL).
- Consider what facts your existing lemmas, function calls, and `broadcast use` statements establish and where the gaps are with respect to your target property.
- Check Verus's output for recommends-failures and other notes.
- By default (i.e., as long as the attribute `#[verifier::loop_isolation(false)]` isn't present), loops do not import their surrounding context, so all necessary facts need to be captured in loop invariants.  After a loop, the only thing Verus knows about what happened inside the loop is the loop invariants and that the loop condition no longer holds.

## Repair the proof
- Does your proof obligation involve non-linear arithmetic or bitwise operations?  If so, you should very likely use one of Verus's dedicated solvers, e.g., `assert(e) by (nonlinear_arith) requires ... { }` or `by (bit_vector)` or `by (integer_ring)`.  These solvers **do not** see their surrounding context, so all necessary facts must be "imported" via the `requires` clause.
- Does your property need an unusual amount of unfoldings of a recursive definition?  If so, you may need more fuel.  Try `reveal_with_fuel(my_function, N)` to unfold the defintion `N` times (the default is 1).
- Is your proof "obviously" true if you just compute on the values (e.g., `pow(2, 8) == 256`)?  Use `assert(e) by (compute)`.
- Does your proof rely on the equality of a container type (like Seq or Map)? Try extensional equality, e.g., by adding `assert(seq1 == seq2);`.
- Think about trigger selection: What instantiations does the proof need?  What terms are missing?  If a quantifier doesn't have an explicit trigger, use `--triggers` to see which terms Verus selected.
- Search vstd before writing your own lemmas.
- If a proof succeeds in isolation (e.g., when run with `--verify-function` but fails in a larger context), try adding the `#[verifier::spinoff_prover]` attribute to the function, to give it its own solver context.

### If the error mentions "rlimit exceeded":
- Try the quantifier profiler (`--profile`) to identify a problematic trigger-pattern.
- Try breaking the proof into smaller pieces:
    - Move a subproof to a lemma
    - Divide a proof into parts that are invoked from a top-level lemma
    - Hide local proofs with `assert(e) by { ... }`
- Invoke one of Verus's specialized solvers instead of relying on SMT
- Use `#[verifier::opaque]` to hide expensive function definitions, and then
  only `reveal(f)` them in limited scopes where they're needed.  Use `hide(f)`
  to locally hide a definition, instead of making it globally opaque.

## IMPORTANT NOTES
- **DO NOT** increase the resource limit (rlimit) for a proof via `#[verifier::rlimit(...)]`.  
  This rarely succeeds, often leads to brittle proofs, and increases the time
  it takes to iterate on the proof.
- Be sure to **remove all assumptions** used while debugging a failed proof.  It has not
  been repaired if you haven't removed those assumptions.
