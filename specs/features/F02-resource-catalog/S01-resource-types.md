# F02 · S01 — Resource Type & Tier Definitions

**Feature:** F02 — Resource & Recipe Catalog
**Scope:** both · **MVP:** yes · **Depends on:** —

## Intent
Define every material the game knows about, each with a stable id and a `Tier`, in one
authored, immutable catalog so resources are data, not code.

## Behavior
1. The catalog holds a list of `ResourceDef { id, name, tier }`. Ids are unique and
   dense (usable as indices). Raw ores have `tier = 0`.
2. MVP resources include at least: `Stone`, `Iron`, `Copper` (tier 0) and the tier-1+
   products named in `feature.md`.
3. The catalog is loaded once and immutable at runtime; no command mutates it.
4. `Terrain` resource kinds (`Stone/Iron/Copper`) map to catalog tier-0 resources.

## Data touched
`ResourceDef`, `Catalog.resources`.

## Acceptance criteria
- **AC-1** *Given* the loaded catalog, *then* all resource ids are unique and every
  ore terrain maps to a tier-0 resource.
- **AC-2** *Given* any `ResourceId` used elsewhere, *then* it resolves to a
  `ResourceDef` (Invariant I5).
- **AC-3** *When* a new resource is appended, *then* no other feature spec changes.

## Out of scope
Recipes (S02); how resources are produced/moved (F06–F08).
