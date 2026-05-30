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

## Acceptance criteria
- **AC-1** *Given* two entities matching a system's query, *then* the system
  processes them in ascending-id order (or the declared key) on every run.
- **AC-2** *When* the schedule runs, *then* systems execute in exactly the order of
  `01-ecs-model.md` §5; reordering is observable and disallowed.
- **AC-3** *Given* a presentation-only component is mutated, *then* no
  simulation-affecting state changes as a result (isolation holds).
- **AC-4** *Given* identical component data, *then* a system produces identical writes
  regardless of internal storage layout.

## Out of scope
The behavior of individual systems (owned by F05–F08); parallel execution details
(allowed only if result-identical to serial, §9 of the LOD spec).
