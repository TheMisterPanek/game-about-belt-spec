# F01 · S02 — ECS World, Components, and Ordered System Schedule

**Feature:** F01 — Simulation Core & ECS
**Scope:** both · **MVP:** yes · **Depends on:** S01

## Intent
The abstract ECS substrate: entities, plain-data components, and systems running in a
fixed order. Other features attach their components and register their systems here.

## Behavior
1. The ECS world stores entities (opaque ids) and components (plain data) per
   `01-ecs-model.md`. A component type attaches at most once per entity.
2. Systems declare a query (required/excluded components) and run as `System(world,
   Δt)`. They are the only place simulation logic lives.
3. Systems run in the canonical schedule order (`01-ecs-model.md` §5), the same order
   every tick.
4. Within a system, entities are visited in deterministic order (ascending entity id
   unless the system declares another key). No result-affecting iteration over
   unordered collections.
5. Component fields that affect simulation are integer/fixed-point; presentation-only
   real-valued components never feed back into systems.

## Data touched
All component families (`01-ecs-model.md` §3); the system schedule (§5).

### Entity ID assignment
Entity IDs are globally unique 64-bit integers, assigned monotonically:
- The first entity spawned gets `id = 1`, the second gets `id = 2`, etc.
- IDs never wrap, reuse, or go backward.
- When an active cell is demoted (F01 S06), its micro entities are deleted; their 
  IDs are consumed and never reused.
- When a cell is promoted again, its micro entities are created with new IDs 
  (the next available from the global counter).

**Why:** Ascending ID iteration is stable and deterministic given the command 
sequence. If systems iterate entities in ascending ID order, they process them in 
creation order, which is deterministic and reproducible.

## Acceptance criteria
- **AC-1** *Given* two entities matching a system's query, *then* the system
  processes them in ascending-id order (or the declared key) on every run.
- **AC-2** *When* the schedule runs, *then* systems execute in exactly the order of
  `01-ecs-model.md` §5; reordering is observable and disallowed.
- **AC-3** *Given* a presentation-only component is mutated, *then* no
  simulation-affecting state changes as a result (isolation holds).
- **AC-4** *Given* identical component data, *then* a system produces identical writes
  regardless of internal storage layout.
- **AC-5** *When* a cell is promoted, *then* its micro entities are assigned new, 
  monotonically increasing IDs (never reused from before demotion).

## Out of scope
The behavior of individual systems (owned by F05–F08); parallel execution details
(allowed only if result-identical to serial, §9 of the LOD spec).
