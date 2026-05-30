# F06 · S02 — Single-Resource Belt Transport

**Feature:** F06 — Micro Logistics: Miners, Belts, Power
**Scope:** micro · **MVP:** yes · **Depends on:** F05 S02

## Intent
Move items across the floor on directional belts that carry **one** resource type
(CONSTITUTION Art. VII) — one type per belt, never two interleaved lanes.

## Behavior
1. A `Belt` has a `dir`, a single carried `resource` (or empty), and a fixed-length
   ordered `slots` track. `BeltSystem` (schedule step 7) advances items from the input
   edge toward the output edge at the belt's speed.
2. **Single-resource rule (Invariant I1):** a belt's `resource` may be set only when it
   is empty. A source attempting to put a *different* resource onto a non-empty belt is
   back-pressured (refused), never merged. Same-resource items pack normally.
3. Items advance in a deterministic per-belt order; a belt whose output is blocked
   back-pressures upstream so items do not overlap or vanish.
4. All movement is `Fixed`/integer; no item is created or lost in transport.

## Data touched
`BeltState` (dir, resource, slots).

## Acceptance criteria
- **AC-1** *Given* any sequence of feeds, *then* a belt never simultaneously holds two
  distinct resources (Invariant I1).
- **AC-2** *Given* a different resource offered to a non-empty belt, *then* it is
  refused (back-pressured at the source).
- **AC-3** *Given* a blocked output, *then* items back up upstream without loss or
  overlap; when unblocked, flow resumes.
- **AC-4** *Given* identical layout and feeds, *then* item positions advance
  identically tick-by-tick across runs (determinism).

## Out of scope
Junction hand-off between belts (S03); export at outputs (F05 S03).
