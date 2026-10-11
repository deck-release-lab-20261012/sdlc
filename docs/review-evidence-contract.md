# Review expectations and evidence comparison

A review can approve the right code while using the wrong model or reasoning
level. A model label in a verdict cannot establish what actually executed.
SDLC supplies a policy expectation and a deterministic comparison component;
the launcher, Review and consuming merge gate retain the duties that make the
observations trustworthy.

## Build from the adopted policy

```sh
noise sdlc review request --target /path/to/adopted-project --input candidate.json
```

The input identifies `requestID`, `repo` (owner/name), numeric `pr`, full
`baseSHA` and `headSHA`, `diffDigest` (`sha256:` plus 64 lowercase hex digits),
`harnessVersion`, `expectedBot`, and `contributors`. Each contributor has a
cataloged `harness`, `model` and public `evidenceRef`. Include every actual agent
contributor, including repair agents. Public references are locators, never
transcripts, credentials, environment dumps or hidden reasoning.

The constructor verifies the adopted snapshot and pinned catalog, selects the
existing `review.formal` binding and requires its declared version to have a
review invocation contract. It derives family and reasoning from the binding
and catalog. Unknown contributors, a same-family contributor, unsupported
review invocation or stale catalog prevents construction. It never substitutes
a different route, probes a harness or launches a provider.

The output contains an `expectation` object and explicitly labels candidate
identity as caller-declared and execution authority as not granted. The
expectation binds request/repository/PR, base/head/diff, policy/catalog digests,
the selected reviewer, contributor declarations, expected bot and full coverage.
The constructor cannot prove that the caller named the real candidate, supplied
every contributor, chose the correct Review-owned bot identity or observed the
installed version. Those remain external-owner evidence obligations.

An existing self-adoption may still pin an older catalog; the default Codex
review route also lacks a versioned invocation action in the current catalog.
In either case this command refuses a request. Use a reviewed adoption/update
with a supported route to exercise successful construction; do not silently
rewrite the receipt or change a model to make the example succeed.

## Compare separate observations

```sh
noise sdlc review compare review-evidence.json
```

The input follows [review-evidence.schema.json](schemas/review-evidence.schema.json):
`schemaVersion: 1`, an `expectation`, and `observations`. Parsing is bounded to
1 MiB, rejects unknown fields, duplicate JSON keys (including escaped aliases),
trailing data and unsupported versions, and preserves integer IDs exactly.
Evidence references are never opened. Valid JSON is not authenticated evidence.

| Observation | What the external owner must establish |
|---|---|
| Current base/head | The candidate still being considered by the consuming gate |
| Execution | Request and candidate association, observed harness/version, requested and effective model/reasoning, and a launcher evidence reference |
| Publication | Request/repository/PR association, exact review ID, bot identity/type, state, published commit and authoritative readback reference |
| Coverage | Request/base/head, expected versus delivered and reviewed diff digests, full delivery, no truncation, declared complete review, and unresolved findings |
| Optional audit | The parsed pointer from invocation output; when present it must agree with authoritative publication observations |

Omit an observation group that has not been obtained. Empty current base/head
means unknown; null effective model/reasoning means unknown. Explicit false
coverage flags and nonzero unresolved findings are contrary evidence, not
missing values. The schema requires those fields so absent data cannot quietly
decode to a favorable zero or false.

The comparator checks exact requested and effective settings; higher-looking
reasoning is not assumed equivalent. Model families come from the pinned
catalog, including every declared contributor. A Codex repair contribution can
therefore conflict with a Codex reviewer even if the original implementation
route was different. Unknown or lower-capability reviewer routes fail the
comparison.

Head/base/request identities must agree across their respective observations.
Expected, delivered and reviewed diff digests must agree independently of a
matching head. Full delivery does not by itself prove complete review. The
publication must identify the expected bot, an approved current verdict and the
expected candidate. An optional audit cannot override contradictory readback.
Missing audit output can be reconciled through request-bound owner readback;
it is not an instruction to repost.

Output `status` is `match`, `mismatch` or `incomplete`, with sorted stable reason
codes. Contradictions take precedence over missing observations. The CLI emits
the comparison and exits nonzero for mismatch or incomplete evidence. Even a
match always reports:

```json
{
  "approvalProof": "not-established",
  "mergeReadiness": "not-assessed",
  "authenticity": "not-assessed"
}
```

A caller can manufacture matching JSON. A trusted consuming gate must validate
the expectation's adopted-policy origin, authenticate and bind each owner
observation, establish the complete contributor roster, and check current
candidate and authorization again immediately before any merge. This component
does not add that gate, a round registry, a signature system or a trusted
launcher. It does not read GitHub, post a verdict, retry, merge, close a task or
transfer work ownership.

## Failure evidence and remaining acceptance

An approved verdict from a different model or reasoning level cannot satisfy
the selected review policy. Acceptance requires evidence of what actually
executed, not just the intended route or the verdict's model label.

Synthetic tests reproduce lower-model substitution, wrong harness/version/model
or reasoning, forged family, repair-family overlap, stale head/base/request,
wrong PR/review/bot, withdrawn approval, mismatched diff coverage, unresolved
findings, missing observations and missing audit reconciliation. These tests
prove comparison behavior, not that a real review completed.

Shipping this component does not complete the acceptance boundary. Full
acceptance requires a trusted producer for actual reviewer provenance and a
consuming merge gate that rejects absent or mismatched provenance. Review
publication and dispatch work must be verified at its delivered revision;
recorded task status or a working branch is insufficient. Race, coverage and
publication integration require evidence from their external owners. Controller
ownership and standing authority remain separate architecture decisions; SDLC
owns no round-state registry.
