# Contributing

Thanks for your interest! This is an unusual repository: it is a **specification-only**
project (a game design that doubles as a cross-language AI benchmark). Contributions are
**spec changes**, not code. Please read `CONSTITUTION.md` first — it is the supreme law
here and overrides anything below.

## The golden rule

**No implementation code in this repository.** Not in any language, not as an
"example." If something seems to need code, it is really a request to *amend a
specification* (CONSTITUTION Art. II). Implementations live in *your own* repo, built
*from* these specs.

## What you can contribute

- **Clarifications & fixes** to existing specs (ambiguity, contradictions, broken
  cross-references).
- **New stories** under an existing feature, following the template.
- **New features** (a new `Fxx-<slug>/` folder) — discuss in an issue first; features
  must map to a module (`specs/architecture/04-module-boundaries.md`).
- **Catalog content** (new resources/recipes) via F02 — never hard-coded elsewhere.
- **Benchmark scenarios** (`benchmark/scenarios.md`) and scoring refinements.
- **Constitution amendments** — only via a dated entry in `CONSTITUTION.md` §X, with a
  clear rationale, and only when no lower-level change suffices.

## House rules (enforced in review)

1. **Language/engine-neutral.** No programming language, engine, library, or
   rendering-API references. Express data in the neutral pseudo-notation of
   `specs/architecture/02-data-model.md` (CONSTITUTION Art. III).
2. **Use the glossary.** Only terms from `GLOSSARY.md`, with their exact meanings. New
   term? Define it in the glossary in the same change.
3. **Determinism & fixed-point.** Anything affecting simulation state is integer/`Fixed`
   (scale 1/1024), never `Real` (CONSTITUTION Art. IV).
4. **Single-resource logistics (Invariant I1).** Belts and roads carry one resource type
   at a time — this is law, not a tunable.
5. **Stories follow the template** in `specs/README.md`: Intent → Behavior → Data
   touched → Acceptance criteria → Out of scope. Every `AC-n` must be checkable from
   observable state/dump output, not from reading source (CONSTITUTION Art. IX).
6. **IDs are immutable.** Never renumber or reuse `Fxx` / `Syy` / `AC-n` / `Fxx-Ay` —
   the benchmark cites them. Deprecate instead of deleting.
7. **Single source of truth.** Cross-reference (`see F01 S06`, `Invariant I1`) rather
   than restating.

## Submitting a change

1. Open an issue describing the gap or proposal (especially for new features or
   Constitution amendments).
2. Make the change as a branch/PR. Keep it focused — one capability or fix per PR.
3. In the PR description, note: which `Fxx`/`Syy` it touches, whether it adds glossary
   terms, and any benchmark scenario it adds or affects.
4. Self-check against the **House rules** and the **Definition of Done** in
   `specs/README.md` before requesting review.

## Checklist before opening a PR

- [ ] No source code added (any language).
- [ ] No language/engine/library/rendering references.
- [ ] Only glossary vocabulary (new terms defined in `GLOSSARY.md`).
- [ ] Simulation-affecting values are integer/`Fixed`, not `Real`.
- [ ] New/changed stories follow the template with checkable `AC-n`.
- [ ] No IDs renumbered or reused.
- [ ] Cross-references updated (feature index, data model, glossary, roadmap as needed).

## License

By contributing, you agree that your contributions are licensed under the project's
[MIT License](./LICENSE).
