# ADR 0003: Keep the catalog closed while accepting evidence and implementation

**Status:** Provisional; revisit immediately before public release
**Date:** 2026-08-12

## Context

If this repository becomes public, collaborators may submit pull requests that
add their own SDLCs or alter the normative policies in existing collections.
The project exists to curate collections from actual operated work, not to host
a general policy marketplace. Accepting a collection implies endorsement,
lineage placement, migration support, documentation, and continuing ownership.

At the same time, rejecting every outside contribution would discard useful
implementation fixes, defect reports, and evidence about how the collections
behave outside their original projects.

## Proposed public-release decision

Adopt a **closed catalog, open evidence and implementation** posture.

- Normal implementation, safety, test, and non-normative documentation pull
  requests may be reviewed normally.
- Reproducible defect reports and bounded field evidence are welcome.
- Unsolicited pull requests that add a collection, create a lineage, create a
  convergence release, or make a normative policy change are closed without
  merging by default.
- An external policy idea begins as a curation proposal describing real use,
  the observed gap, and why project-local additions or an existing lineage are
  insufficient.
- A normative pull request proceeds only after a maintainer accepts ongoing
  ownership, accepts the curation proposal, and requests the implementation.
- A new collection or lineage ships only after a project-owned canary has used
  it and maintainers accept its migration and support surface.
- External forks are allowed but receive no namespace, compatibility, upgrade,
  support, or future-inclusion promise from this catalog.
- Runtime lineage provenance names exact releases in this catalog. External
  influence is credited in curation notes rather than represented as a false
  adoptable ancestor.

## Rationale

The default is intentionally asymmetric. Code fixes can be bounded and tested;
an SDLC encodes judgment across projects and creates an indefinite product
obligation. Requiring prior curation prevents a large pull request from using
review effort as a back door to catalog inclusion.

This posture also preserves learning. Contributors can show where a curated
collection failed or what a real project required without asking this project
to adopt their whole operating model.

## Consequences

- Contribution guidance must state before submission that unsolicited
  normative pack pull requests are not accepted.
- Maintainers may close such pull requests without reviewing their policy
  merits and redirect authors to the evidence-first path.
- Maintainer sponsorship means accepting ongoing ownership, not merely granting
  permission for someone else to maintain an upstream pack indefinitely.
- Trust, popularity, or completeness does not bypass curation.
- The project does not need a community namespace, remote registry, trust
  ranking, signature system, or third-party compatibility promise in v1.
