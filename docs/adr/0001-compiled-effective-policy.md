# ADR 0001: Compile one effective policy into agent, human, and machine views

**Status:** Provisional canary hypothesis; not yet a public contract
**Date:** 2026-08-12

## Question

Should lifecycle policy live directly in `AGENTS.md`, in a standalone
`SDLC.md`, or in some combination that remains safe across agent harnesses and
maintainable across many repositories?

## Candidate decision

Test a compiled effective-policy architecture:

1. An immutable lifecycle pack is the sole upstream reusable base.
2. `.sdlc/project.yaml` is the only project-editable lifecycle input. Its
   namespaced additions may strengthen or specialize the base but cannot
   override or delete it.
3. `SDLC.md` is a complete, whole-file-generated human and agent projection
   of the pinned pack plus the validated project input.
4. A managed block at byte zero in root `AGENTS.md` tells agents to load
   `SDLC.md`, repeats the compact critical-safety kernel, states precedence,
   and fails closed if authority cannot be verified.
5. `.sdlc/adoption.json` binds the exact pack input, project input, combined
   effective policy, renderer schema, and rendered outputs by digest.
6. Specialist tools and project systems may implement the collection's
   requirements, but remain separate owners. Their enforcement state is not
   part of the adoption receipt or collection-integrity verdict.

Generated repetition is allowed; hand-maintained duplicate normative prose is
not. Nested `AGENTS.md` files may add scoped domain or operational detail, but
lifecycle additions must enter through `.sdlc/project.yaml`.

## Safety requirements under test

- Load failure permits read-only inspection and remediation planning only. It
  blocks edits, approvals, merges, publication, and any plan that assumes the
  lifecycle was loaded.
- Project rule IDs use `project.<project-slug>.*`, are collision checked, and
  declare `provider`, `manual`, or `advisory` assurance. Provider-owned rules
  name an external owner and contract; this project does not judge its result.
- `SDLC.md` carries an exact end marker that binds its effective-policy
  digest.
- Verification requires exactly one root bootstrap block, root `SDLC.md`, and
  root receipt; exact markers and matching digests; and no shadow lifecycle
  artifacts.
- Adoption and update render and validate the whole staged tree before
  installation, then install transactionally with the receipt last as the
  commit marker.
- Collection integrity must never be reported as lifecycle conformance or
  merge readiness; specialist tools own those judgments.

## Design rationale

The document-authority alternatives are:

- complete generated `SDLC.md` plus a loader-only `AGENTS.md` block;
- complete generated `SDLC.md` plus a loader and critical-safety kernel; and
- a complete bounded policy directly in `AGENTS.md`, due to reference-loading
  risk.

The candidate retains a complete effective policy and addresses reference-loading
risk with a byte-zero loader plus generated critical kernel. It distinguishes
base from effective authority, namespaces local rules, binds all inputs and
outputs, makes bootstrap failure fail closed, prohibits nested lifecycle
overrides, verifies unique roots and exact markers, and makes adoption
transactional.

The proposed human-led `noise/core@1` pack retains the same structural contract
as agent-led packs: core renders a minimal loader, precedence, integrity, and
fail-closed block, but no agent-workflow requirements. This protects incidental
or future agent access without changing core's human-led substance.

This architecture remains an incubation experiment. The first
canaries must compare its clarity and maintenance cost with a simpler
`AGENTS.md`-only result.

## Consequences

- An agent sees the mandatory entry point where harnesses already discover
  repository instructions, while humans get a coherent standalone lifecycle.
- The full effective lifecycle is inspectable without interpreting a receipt
  or merging prose mentally.
- Projects can add real local requirements without forking a pack or silently
  weakening shared rules.
- Deterministic regeneration removes most policy drift, while digest and shape
  checks expose truncation, stale projections, and shadow copies.
- If retained, the compiler, schemas, precedence rules, and transactional
  installer become core product surface and require strong fixture and
  failure-injection tests.

## Rejected alternatives

### Put all lifecycle prose only in `AGENTS.md`

This maximizes discovery but mixes generated reusable policy with
project-specific agent instructions, makes human consumption worse, and
encourages hand-edited forks.

### Put all lifecycle prose only in `SDLC.md`

This is clean for humans but depends on every harness and agent following a
reference before taking action. A missing or ignored link becomes a silent
safety failure.

### Keep independently editable policy in both files

Two hand-maintained normative sources inevitably drift. Any necessary
repetition must be generated from the same resolved policy and verified by
digest.
