# Incubation scorecard

This scorecard decides whether `noise sdlc` should continue, reshape, publish,
or be killed. One bounded JSON record is written for each authorized canary
using the version-2 [`evidence.schema.json`](schemas/evidence.schema.json). Records contain
counts, public identifiers, diffs, and short field notes only. They must not
contain transcripts, prompts, hidden reasoning, credentials, authentication
output, account identity, balances, quotas, environment values, or private
policy copied from another project.

## Per-canary measures

| Area | Required observation |
|---|---|
| adoption | elapsed minutes, consequential questions asked, recommendation accepted/adjusted/rejected, override count and bounded reasons |
| policy | reusable rules accepted, moved project-local, or removed; local rule count; duplicate/conflicting policy count |
| ownership | specialist-tool references tested, actionable references, authority-confusion incidents |
| integrity | generated drift findings and false positives |
| update | elapsed minutes, conflicts, preserved local rules, rollback result |
| outcomes | prevented and missed lifecycle failures, each as a bounded non-sensitive note |
| inventory | installed harnesses, matched/unverified versions, enumerated and unknown capabilities, probe failures |
| routing | accepted/replaced bindings and adequacy for bulk, standard, high-risk, formal-review, and repair roles |
| escalation | escalation count, trigger round, destination tier, and whether review converged |
| compatibility | harness argument drift, stale catalog facts, and Deck interoperability gaps |
| intervention baseline | declared observation window and cohort, accepted-task denominator, avoidable interventions, necessary product/authority decisions, unclassified actions, and coverage |
| delivery timing | elapsed milliseconds partitioned into queue wait, execution, human-decision wait, and unattributed time; human labor recorded separately with its method |
| cleanup | post-cleanup observation time, retained workspace bytes, and artifact count whose ownership remains unknown within the canary boundary |
| provenance | bounded public source references, source revision or explicit unknown, and limitations for every reported measurement |

`unknown` is a valid result and must not be converted to zero, unavailable, or
successful. A missing observation makes the record incomplete.

## Operational evidence contract

Version 2 requires an `operations` object even when nothing was measured.
Every field is present; counts and durations are nonnegative integers,
`unknown`, or `not-applicable`. Zero means an observed absence, while
`not-applicable` means the measure does not apply to the declared scope.
Neither is a substitute for missing evidence. Window and intervention
coverage are separately `complete`, `partial`, or `unknown`.

The observation window uses RFC3339 timestamps, `unknown`, or `not-applicable`.
Its bounded public cohort label and inclusion rule identify the work being
observed. Define inclusion before comparing results: retain incomplete,
failed, interrupted, and cancelled work in the cohort and describe its
disposition in the source record. Do not select only successful work after
seeing the outcome. The denominator counts distinct accepted tasks in that
cohort, backed by the project acceptance owner's evidence; a task status alone
does not prove acceptance.

Count each distinct operator action once. Avoidable interventions include
corrections, restated existing decisions, repeated setup instructions, and
manual relays or recovery that the accepted workflow should have handled.
Necessary product or new-authority decisions are separate. Keep ambiguous
actions unclassified rather than forcing them into a favorable category.
An intervention rate is unavailable when its denominator is zero or unknown,
its observation coverage is incomplete, or unresolved classification prevents
an honest comparison. Heuristic chat matches, selected historical cases,
conversation files, and A2A acknowledgements are not verified incidents or an
accepted-task denominator. Do not report chat-hit incident rates.

Timing uses `per-task-exclusive-elapsed-sum`: partition each observed task's
elapsed interval into mutually exclusive queue wait, active execution,
human-decision wait, and unattributed time, then sum the same cohort's task
intervals. Include interrupted work only through the stated observation end.
This aggregate can exceed the calendar observation window when tasks overlap;
it is not fleet wall-clock duration. Queue wait covers waiting for work or
proof capacity, execution covers active work or proof processing, and
human-decision wait requires an identified blocking decision request. Record
unexplained time as unattributed. Fully measured components must add up to the
reported elapsed total; partial measurements cannot establish that partition.

Human-decision waiting is elapsed blocking time, not human labor. Human labor
is a separate millisecond measure supported by `direct-observation` or an
`operator-report`, with the source and method stated. Otherwise its value and
method remain `unknown` or `not-applicable`. Never infer human hours from
message gaps, provider duration, overnight activity, or elapsed time minus
queue wait. Existing adoption/update elapsed minutes retain their original
meaning and are not converted into labor.

Cleanup observations cover `canary-owned-workspaces` at the recorded time.
Retained bytes are logical file bytes within the owner's declared workspace
boundary after cleanup, counted consistently without following symlinks;
they are not whole-volume usage. Count distinct artifacts within that boundary
whose ownership could not be established, without copying their contents or
private names. Unknown ownership is not deletion authority. Missing workspace
accounting yields unknown values, not a zero-growth claim.

Numerical operational measurements and observed cleanup timestamps require a
nonempty `sourceRefs` list naming entries in `operations.sources`. Sources use
a bounded ID, one of `artifact`,
`owner-evidence`, or `operator-report`, a public repository-relative path under
`docs/` or `evidence/`, and a full Git revision or `unknown`. Refer to bounded
project evidence, not private transcript stores or provider output. Source
IDs and cohort labels are public tokens of at most 128 characters; each
source/reference list has at most 64 entries. At most 16 limitations of up to
280 characters describe missing coverage, methods, and other constraints.
Unknown measures may have empty source references and an explicit limitation.
`privacyReviewed: true` records a declaration; it does not prove the source is
complete, private-data-free, or authoritative for the claim.

The read-only check is:

```text
noise-sdlc check-evidence evidence/sdlc-self.json
```

Schema and consistency validation do not approve a canary or milestone. An
all-unknown operational baseline can be valid while remaining insufficient to
evaluate a gate. The provisional intervention-reduction target requires a
comparable measured baseline and follow-up; it cannot be evaluated from the
historical screening totals.

## Preserved baseline and version history

The original strict v1 contract remains in
[`evidence-v1.schema.json`](schemas/evidence-v1.schema.json). It does not accept
operational fields or the rollback value `not-tested`. The historical
self-canary record used `not-tested` outside that v1 enum. Version 2 explicitly
permits the value, preserving the recorded lack of testing rather than
inventing a rollback result or weakening the historical schema.

The shipped [example record](../evidence/sdlc-self.json) is synthetic: every
measure is `unknown` with explicit limitations, demonstrating a valid
all-unknown baseline without reporting any real adoption, cohort, or
measurement. Its `not-tested` rollback value is deliberate — permitted here
and rejected by the v1 schema — so the record also exercises the v1/v2
boundary described above. A real canary record carries bounded observations
under the privacy rules above; nothing is inferred from absent measurements.

An all-unknown record validates without approving anything: the schema
extension collects no runtime evidence by itself and approves no milestone,
continuation, or publication decision.

## Experiment gates

Continue only when all of these are true after three authorized canaries:

- at least three reusable rule IDs survive unchanged in every canary;
- at least two canaries accept the recommendation with at most two routing or
  lifecycle adjustments;
- the median guided flow asks no more than five consequential questions;
- local additions are fewer than reusable rules in every canary;
- at least one seed or catalog update is applied to two canaries with preserved
  local policy and a working rollback;
- at least one repeated manual harness/model routing instruction is eliminated
  in every canary;
- every formal-review binding is model-family independent from its applicable
  implementation binding, or is explicitly unresolved;
- at least one live-certified harness contract either survives a real CLI
  version change or fails closed before dispatch; and
- no canary confuses collection integrity with review, security, runtime, or
  merge-readiness evidence.

Reshape when the core policy or routing is useful but any of these occur:

- the median flow needs more than five questions;
- two or more canaries replace more than two recommended bindings;
- the generated `SDLC.md` plus receipt causes more authority confusion than a
  loader-only `AGENTS.md` experiment;
- inventory is useful but adoption/rendering is not, suggesting a Deck-facing
  catalog tool; or
- deterministic rendering is useful but a guided skill is a clearer interface
  than a standalone command.

Kill the experiment when any of these occur:

- fewer than three reusable rule IDs survive across the three canaries;
- two canaries reject the recommendation or still require manual routing for
  most roles;
- the update costs at least as much as reconstructing policy manually in both
  update canaries;
- safe probes cannot provide enough capability information to improve routing;
- the ownership boundary repeatedly causes a false conformance, review,
  security, or spend-authorization claim; or
- operating the catalog requires credential/config inspection or provider
  execution outside the accepted boundary.

Publishing remains separately gated by the charter. Passing this scorecard is
not publication authorization.

## Decision record

The continuation review writes `evidence/decision.json` with one outcome:
`continue`, `reshape`, `publish-candidate`, or `kill`. It cites the three
canary record paths, the two update record paths, each gate as pass/fail/unknown,
and a short operator-authored rationale. `publish-candidate` means only that
the evidence permits asking for publication authorization.
