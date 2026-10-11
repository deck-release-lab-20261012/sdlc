# ADR 0009: Refresh current Kimi and GLM execution routes

**Status:** Accepted
**Date:** 2026-08-28

## Context

The installed inventory moved beyond the catalog's one-patch compatibility
ranges. Kimi 0.39.0, OMP 18.0.7, OpenCode 1.18.23, and Pi 0.84.3 were therefore
visible but ineligible. The old balanced template also routed GLM 5.2 even
though bounded OMP and OpenCode enumeration now exposes GLM 5.3 selectors.

Inventory presence is not certification. The refresh must preserve exact
version and argument contracts, distinguish enumerated selectors from operator
declarations, and retain independent formal review.

## Decision

Publish `noise/harnesses@0.4` with `deck/balanced@0.2` as the bundled catalog
and execution template.

- Match only the exact installed patch versions named above.
- Catalog Kimi through `kimi-code/k3`. The managed model's documented default
  is high effort, but the Kimi CLI contract does not expose a per-invocation
  reasoning flag. Kimi therefore remains inventory-visible but is not a
  candidate for roles whose required effort must be enforced at launch.
- Route GLM 5.3 through the exact harness selectors observed or documented:
  `zai/glm-5.3` for OMP and Pi, and `opencode-go/glm-5.3` for OpenCode.
- Admit GLM 5.3 to high implementation and formal review while preserving the
  different-model-family constraint. Admit it to bulk and standard
  implementation behind the existing ordered candidates.
- Keep repair on Codex and Claude because that role requires `xhigh`, which the
  newly promoted routes do not provide.
- Leave Kilo and Devin's unobserved GLM 5.2 descriptors unchanged. They are not
  execution-template candidates and cannot become routable merely because a
  different harness enumerated a newer model.

## Evidence boundary

Exact-argv, fake-CLI inventory, model provenance, resolver, and candidate-order
tests cover the refreshed routes. OMP and OpenCode selectors are bounded safe
enumeration results; Kimi and Pi model availability remains operator-declared.
All ranges remain experimental and make no claim of live provider, spend,
interactive Deck, or secure-mode certification.

## Consequences

Existing answers and adopted policy pinning `deck/balanced@0.1` must be updated
explicitly. Deck must accept the new catalog and template digests before it can
consume them. No provider process is launched by this decision.
