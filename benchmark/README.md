# Benchmark

How to use this spec set to evaluate an AI model's implementation of the game, in any
language or engine. The specs are the prompt; this directory is the rubric.

## The protocol

1. **Input to the model:** the whole `specs/` tree, `CONSTITUTION.md`, `GLOSSARY.md`,
   and a chosen target (language/engine). The model implements the game.
2. **Required seam:** the implementation must expose the **headless harness** of
   `F01 S07` — generate a world from `WorldConfig`, apply an ordered
   `(tickNo, command)` script, tick N times, and emit a structured **state dump** of
   logical state at requested ticks. Scoring is automated against this dump; no GUI is
   required to score.
3. **Run the reference scenarios** (`scenarios.md`) and compare dumps to the expected
   invariants and totals.
4. **Score** against `acceptance.md`.

## What "correct" means here

> This file is the **external scoring rubric**. The implementor's **internal** testing
> methodology — how to step the headless core and assert invariants per tick while
> building — is `specs/architecture/05-testing-strategy.md`. The two are mirrors: its
> properties and per-milestone gates map onto the axes below.

Because the specs are language-neutral and the simulation is deterministic and
fixed-point (`CONSTITUTION` Art. III–IV), two correct implementations in two languages
must produce **equal logical state at equal `tickNo`** on the same input. The benchmark
exploits this: it checks determinism, conservation, and the LOD losslessness property
**P-LOD** directly from dumps — properties that are hard to fake without actually
building the system correctly.

## State dump contract

A dump at a given `tickNo` is a deterministic, structured listing of logical state:
per-cell terrain/occupancy, `MacroOutput.buffer`s, road/manufacturer buffers, valid
profiles, deposits remaining, and `tickNo`. It **excludes** presentation (camera) and
transient micro entities (reconstructable). Two dumps are compared for equality
field-by-field; floating/real values must not appear (they would indicate a
determinism violation).

## Scoring axes (see acceptance.md for weights)

- **Determinism** — identical dumps across runs/machines for identical input.
- **Conservation** — material is never created or destroyed (exact).
- **LOD losslessness (P-LOD)** — coarse vs fine totals agree within ε; the signature,
  highest-weight axis.
- **Single-resource invariant** — belts/roads never mix resources (I1).
- **Feature acceptance** — each feature's `Fxx-Ay` and story `AC-n` criteria.
- **Spec fidelity** — no language/engine leakage into observable contracts; outputs
  follow the data model and glossary.

## Cross-language fairness

The same scenario scripts and expected invariants apply to every target. A target's
idiomatic data structures, rendering, and concurrency are free (CONSTITUTION Art. III)
as long as the dumps match. This makes the benchmark a comparison of *reasoning and
correctness*, not of language-specific boilerplate.
