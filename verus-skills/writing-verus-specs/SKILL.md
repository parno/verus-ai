---
name: writing-verus-specs
description: Write and review Verus requires/ensures clauses and spec fns. Use before implementing a verified function, when adding specs to existing Rust code, or when a spec change is considered.
---

Verus specifications are attached to standard ("exec") Rust functions 
via `requires` and `ensures` clauses.  Simple specifications can be
written directly in those clauses, but for specifications that will
be used more than once, or for more complex specifications, it helps
to write them using one or more "ghost" `spec fn` functions.  This aids
reuse and helps break-up large definitions into a more readable form.

Specifications are also used with `proof fn` functions, which serve as lemmas
that can be invoked in multiple places as needed.

## Tips for Writing Spec Functions
0. Check Verus's standard library, `vstd`, for specs before writing your own.
1. A `pub spec fn` must be marked as either `pub open spec fn` or `pub closed spec fn`.
   Open means the definition is available outside the function's module.  Closed means
   that it's hidden.  Use hidden specifications to define abstraction boundaries.  Code
   inside the module can use the definition to prove postconditions, while callers 
   outside the module don't have their proof context polluted with extraneous details.
   For example, a data type can maintain a complicated well-formedness condition internally,
   but provide an API that just promises to maintain the abstract well-formedness property.
   In general, use `closed` for abstraction, and `#[verifier::opaque]` for
   controlling verification performance.
2. In general, specifications should be written in a concise, easy-to-read style, using the
   simplest, clearest constructs possible.  This is especially critical for "exported" specs,
   i.e., those on the public API, and for "imported" specs, e.g., assumptions introduced
   about external function calls.  A necessary, but not sufficient step, is
   typically to abstract complex implementation types into mathematical types;
   e.g., Verus's vstd library abstracts both `HashSet` and `BTreeSet` as a mathematical `Set`;
   `Vec` as a mathematical `Seq`, etc. The `View` trait makes this style more convenient by
   allowing the use of the `@` syntax as shorthand for `.view()`.
3. "Imported" specs, typically written with `assume_specification` and/or `external_body` are
   *trusted*, so you must tell the user if you add one one of these.
4. Relatedly, the details of the implementation should not (in general) "leak" into the specification.
5. When talking about numbers, use `int` by default, since this is the most
   general type and is supported most efficiently by the SMT solver.  Use nat
   for return values and datatype fields where the 0 lower bound is likely to
   provide useful information, such as lengths of sequences.  Note that int and
   nat are usable only in ghost code.  Arithmetic in spec code is unbounded; e.g., 
   `a + b` on two `u64`s has type `int`.

## Specification review workflow
1. Can the precondition (`requires`) be satisfied?  If not 
   (e.g., `requires false`), the proof is vacuous.
2. Can the postcondition be satisfied by a trivial implementation?  If so, that
   may indicate the postcondition is too weak.  For example, if a function
   returns `Option<T>`, a postcondition that only specifies what happens when
   the return value is `Some(x)` can be trivially satisfied by always returning `None`.
