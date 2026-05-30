# F02 · S03 — Catalog Queries

**Feature:** F02 — Resource & Recipe Catalog
**Scope:** both · **MVP:** yes · **Depends on:** S01, S02

## Intent
Read-only lookups other modules use to stay data-driven: resolve by id, list by tier,
find recipes that produce a resource.

## Behavior
1. Queries: `resource(id)`, `recipe(id)`, `allResources()`, `resourcesByTier(t)`,
   `recipesProducing(resource)`. All are pure and side-effect free.
2. Results are deterministic and ordered (ascending id) so dependent logic is stable.
3. Unknown ids return an explicit "absent" result, never undefined behavior.

## Data touched
`Catalog` (read only).

## Acceptance criteria
- **AC-1** *Given* a valid id, *then* `resource(id)`/`recipe(id)` return the matching
  def; *given* an unknown id, *then* an explicit absent result.
- **AC-2** *Given* tier t, *then* `resourcesByTier(t)` returns exactly the tier-t
  resources in ascending-id order.
- **AC-3** *Given* a resource r, *then* `recipesProducing(r)` returns every recipe
  whose output is r, deterministically ordered.

## Out of scope
Mutation (the catalog is immutable); any simulation behavior.
