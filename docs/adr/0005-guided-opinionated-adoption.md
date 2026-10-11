# ADR 0005: Make adoption guided and opinionated

**Status:** Accepted incubation hypothesis; question set remains provisional
**Date:** 2026-08-13

## Context

Choosing an SDLC from a static list assumes the operator already understands
the relevant lifecycle distinctions, specialist tools, agent-routing choices,
and local project facts. A completely configurable wizard creates the opposite
problem: it transfers policy design to the operator one checkbox at a time.

The product should reduce that cognitive load. It has curated policy knowledge,
a read-only view of the project, and a tested inventory of available harnesses.
Those inputs can produce an explained recommendation while preserving a direct,
reproducible path for experienced operators and automation.

## Candidate decision

`noise sdlc adopt` without an SDLC ID runs a short guided flow:

1. inspect non-sensitive project and harness facts read-only;
2. ask only consequential questions that cannot be inferred safely;
3. recommend one resolved lifecycle and execution profile;
4. show the causal facts, exact policy, role bindings, alternatives, unresolved
   requirements, and complete file plan; and
5. require confirmation before any write.

The tool is a curator, not a policy construction kit. It offers strong defaults
and at most a few materially different alternatives. It does not expose every
rule as a toggle. During incubation, it resolves one seed plus capabilities,
project additions, and an execution profile; it does not fabricate multiple
collections before canary evidence shows stable divergence.

The same planning engine supports direct and noninteractive use. An exact seed
and execution profile may be supplied on the command line, or a non-sensitive
answers file may replay a guided result. For fixed catalog, project facts, and
answers, the recommendation is deterministic.

Committed inputs record the answers that affect policy and the selected public
harness/model identifiers. They do not store the full machine inventory,
account identity, credentials, balances, raw probe output, or hidden inference.
Recommendations are not silently refreshed; later re-resolution is an explicit
diff and adoption decision.

## Consequences

- Recommendation quality and question burden become measurable canary outputs.
- Every question must map to a material policy consequence and be removable if
  it does not change outcomes.
- Project inspection needs a strict allowlist and must distinguish observations
  from operator declarations.
- The planner must explain why one result won and why an alternative differs.
- A headless answers schema and stable JSON plan are part of the eventual
  contract, even though the exact CLI is not yet committed.
- Stable clusters of materially different answers can justify future named
  collections; the question tree must not presume those collections first.

## Rejected alternatives

### Require an SDLC ID for every adoption

This keeps implementation small but leaves the hard policy and model-routing
decisions with the operator—the repeated work the product exists to remove.

### Expose every capability and rule as a question

That is a generic policy builder with high cognitive load and combinatorial
outcomes. Curated defaults and bounded alternatives better match the
Noisefloor microtool ethos.

### Infer everything without asking

Repository and harness observations cannot reveal risk tolerance, spend
preference, publication authority, or human-approval boundaries. Silent
inference would make consequential policy choices opaque.

### Re-resolve automatically whenever the catalog changes

Model availability and pricing are volatile, but changing who implements or
reviews a project is a policy change. It must be explicit, reviewable, and
reversible.
