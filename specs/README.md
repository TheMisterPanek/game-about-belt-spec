# Specs

How the specification is organized and how to write within it.

## Layout

```
specs/
  architecture/      cross-cutting design (read before any feature)
    00-overview.md
    01-ecs-model.md
    02-data-model.md
    03-simulation-lod.md
    04-module-boundaries.md
    05-testing-strategy.md
  features/
    Fxx-<slug>/
      feature.md     the feature: intent, scope, stories index, acceptance
      Syy-<slug>.md  one story: a single user-meaningful capability
```

## Hierarchy

- **Feature (`Fxx`)** — a coherent capability area owned by one module
  (`04-module-boundaries.md`). Has a stable id; never renumbered.
- **Story (`Syy`)** — a single, independently demonstrable slice of a feature, with
  its own acceptance criteria. Stories are the unit of implementation and scoring.

IDs are immutable once published (the benchmark references them). To remove a story,
mark it `Deprecated`, don't reuse its number.

## Story template

Every `Syy-*.md` follows this shape:

```markdown
# Fxx · Syy — <Title>

**Feature:** Fxx — <feature name>
**Scope:** macro | micro | both
**MVP:** yes | no
**Depends on:** <story ids, or —>

## Intent
One paragraph: the capability and why it exists, in user/world terms.

## Behavior
Numbered, precise statements of required behavior. Reference GLOSSARY terms and
data-model types by name. No language or engine assumptions.

## Data touched
Which data-model types / components / commands / systems this story reads or writes.

## Acceptance criteria
Given/When/Then statements that are checkable from observable state or output
(CONSTITUTION Art. IX). Each has an id `AC-1`, `AC-2`, … so the benchmark can cite it.

## Out of scope
What this story explicitly does not cover (forward references welcome).
```

## Writing rules (enforced by review)

1. Use only GLOSSARY vocabulary; new terms get defined in the glossary first.
2. No language, engine, library, or rendering-API references (CONSTITUTION Art. III).
3. Every behavior that affects simulation state must respect determinism and the
   fixed-point contract (CONSTITUTION Art. IV, `03-simulation-lod.md`).
4. Acceptance criteria must be checkable without reading implementation source.
5. Keep stories small enough to demonstrate independently; split if a story has more
   than ~7 acceptance criteria or spans two modules.

## Definition of Done

The same scale applies to every implementation, so runs are comparable. Track status
in `PROGRESS.md`; build in the order of `ROADMAP.md`.

| State | Symbol | Meaning |
|-------|:------:|---------|
| Not started | ☐ | No implementation of this story. |
| In progress | ◐ | Partially implemented; some ACs not yet passing. |
| **Done** | ☑ | **Every `AC-n` of the story passes on the headless harness** (`F01 S07`) via a targeted command script. Behavior is correct in isolation. |
| **Verified** | ✅ | The story's owning **milestone** passes its benchmark scenario(s) in `benchmark/scenarios.md` end-to-end — i.e. the story is correct *in concert* with its slice, including determinism and (where applicable) P-LOD. |

Rules:

- A story cannot be `Done` on the basis of inspection or a passing GUI — `Done`
  requires the headless ACs to pass (CONSTITUTION Art. IX). Real-valued simulation
  state in a dump disqualifies it (gate G2).
- A **feature** is Done when all its non-deprecated stories are `Done` and its
  feature-level `Fxx-Ay` criteria pass.
- A **milestone** is `Verified` only when *all* its exit criteria in `ROADMAP.md` hold;
  a single red exit criterion blocks the next milestone.
- **MVP is complete** when milestones M0–M7 are `Verified` (F10/M8 is Post-MVP).
