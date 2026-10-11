# ADR 0002: Model pack lineages without building a policy VCS

**Status:** Deferred until real project divergence exists
**Date:** 2026-08-12

## Context

Curated SDLCs will begin from shared practice, evolve through real projects,
and sometimes diverge. A useful variant may later contribute ideas back to
another collection. Flat names with unrelated linear versions lose that
history. A full branch and semantic-merge engine would add substantial
complexity before the project has enough real pack evolution to justify it.

Versioning and lineage solve different problems:

- a version identifies an immutable release that a project can pin; and
- lineage records how curated collections evolved and influenced one another.

## Deferred candidate decision

If canary evidence produces durable variants, use a small immutable-release
DAG:

- A stable pack ID such as `noise/agent` names a **lineage**.
- Versions within one lineage advance linearly. A release may name at most one
  `previous` release with the same pack ID.
- A release may name exact immutable releases from other lineages in
  `derivedFrom`. These edges record historical provenance; they do not import
  rules or imply compatibility, and they may not duplicate `previous`.
- Semantic composition is separate and explicit. If a release incorporates
  another pack's rules, it names the exact release under `includes`.
- A convergence release may name multiple `derivedFrom` parents, but its
  resolved policy is authored and reviewed explicitly. The tool does not
  perform an automatic three-way or semantic merge.
- `update` follows the current lineage only. Moving to a different lineage is
  an explicit `switch` with a full diff and ordinary adoption conflict checks.
- Every project receipt pins one exact immutable release and digest. Floating
  aliases are discovery conveniences only.

Example:

```yaml
id: noise/agent-strict
version: 1.0.0
lineage:
  previous: null
  derivedFrom:
    - noise/agent@1.3.0
includes:
  - noise/core@1.2.0
```

A later convergence might declare:

```yaml
id: noise/agent
version: 2.0.0
lineage:
  previous: noise/agent@1.4.0
  derivedFrom:
    - noise/agent-strict@1.2.0
```

The ancestry parents are the same-lineage predecessor and the strict-lineage
derivation. The latter explains influence; it does not cause rules to merge.

## Consequences

- Projects retain reproducible version pins while people can inspect how one
  SDLC variant descended from or converged with another.
- The catalog can show a useful lineage graph and compare any two releases.
- Ordinary upgrades remain predictable because they are linear within a pack
  ID.
- Cross-lineage moves are visibly different from upgrades.
- Provenance, composition, and succession cannot be conflated: `derivedFrom`,
  `includes`, and `previous` have distinct meanings.
- V1 does not need branch pointers, merge bases, mutable heads, conflict
  algorithms, or Git-like commands.
- Optional lineage fields may remain empty until real work creates a fork or
  convergence; the catalog must not fabricate ancestry among the initial packs.

## Rejected alternatives

### Flat names with unrelated linear versions

This is simplest, but it discards the exact historical relationship the
catalog is intended to preserve and makes later convergence look accidental.

### A full branch and merge engine

Policies do not have a safe generic semantic merge. Automatic merging would
create false confidence around conflicts in precedence, rule identity, and
authority. Explicitly authored convergence is safer and sufficient.
