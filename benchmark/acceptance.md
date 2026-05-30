# Benchmark — Acceptance & Scoring

Global acceptance criteria and how implementations are scored. Each axis cites the
specs it verifies. Weights are suggestions; adjust per benchmark run but keep P-LOD
dominant.

## Global gates (must pass to score at all)

- **G1 — Headless harness** (`F01 S07`): the implementation can generate, script,
  tick, and dump without a GUI.
- **G2 — No real numbers in dumps**: every simulation-state value is integer /
  fixed-point (`CONSTITUTION` Art. IV). A real-valued field in a dump fails G2.
- **G3 — Catalog integrity** (`F02-A1/A2`): all ids resolve; no invalid recipes.

A submission failing any gate is unscored (the harness can't be trusted).

## Scoring axes & weights

| # | Axis | Verifies | Weight |
|---|------|----------|:------:|
| A | **LOD losslessness (P-LOD)** | `F01-A2`, `S05`, `S06`, `03-simulation-lod §7` | 30% |
| B | **Determinism** | `F01-A1`, `S01`, `S02`, `S07` | 20% |
| C | **Conservation** | miners/assemblers/roads/mfg conserve material | 15% |
| D | **Single-resource invariant (I1)** | `F06-A1`, `F07-A2` | 10% |
| E | **Micro feature correctness** | F05, F06 story ACs | 10% |
| F | **Macro feature correctness** | F03, F07, F08 story ACs | 10% |
| G | **Blueprints** | F09 story ACs | 5% |

(F10 persistence is Post-MVP; include as a bonus axis when in scope.)

## How each axis is checked

- **A — P-LOD.** Run each scenario twice: once all-Fine (pin every involved cell),
  once with forced promote/demote cycles. Compare cumulative exports over `T = 60 s`:
  must match within ε (±1 unit per resource or ≤1%, whichever larger). Material totals
  must match **exactly**. Any creation/loss fails A outright.
- **B — Determinism.** Run each scenario twice (and ideally on two machines/builds).
  Dumps at every checkpoint `tickNo` must be byte-equal.
- **C — Conservation.** From dumps, verify `produced = consumed + buffered + exported`
  at every checkpoint for every resource; ore extracted ≤ initial deposits.
- **D — Single-resource.** Assert no belt/road dump ever lists two resource types; feed
  adversarial mixed inputs (scenario S-MIX) and confirm refusal/back-pressure.
- **E/F/G.** Drive the specific story ACs via targeted scripts; each AC is pass/fail.

## Pass thresholds (suggested)

- **Bronze:** G1–G3 + B (determinism) + C (conservation) ≥ pass; A may be approximate.
- **Silver:** Bronze + A within 5% on all scenarios + D fully.
- **Gold:** Silver + A within ε on all scenarios + all E/F/G ACs.

## Reporting

For each submission, report: gate results, per-axis score, per-scenario P-LOD delta,
and any invariant violations with the `tickNo` and resource where they first appeared.
Cite failures by spec id (`Fxx-Ay` / `Fxx Szz AC-n`) so results are traceable to the
spec.
