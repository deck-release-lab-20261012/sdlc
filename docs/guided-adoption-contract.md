# Guided adoption and replay contract

`noise sdlc plan` is a deterministic planner. It reads allowlisted project
facts, normalized harness inventory, and four operator answers; recommends one
result; and emits a stable JSON plan. `adopt` may install that exact plan only
after an explicit confirmation or `--yes` on a replayed plan. Planning never
writes project files, invokes a model, contacts a provider, or authorizes
spend.

Deck may offer this workflow after a project is added, but only as a skippable
post-creation card. Project creation never depends on SDLC selection. Starting
selection is explicit, dismissal is remembered per project, and persistent
Deck synchronization requires a second, separate opt-in.
The card is suppressed when the project already has a verified adoption. An
invalid existing adoption enters repair rather than onboarding.

## One-time agent bootstrap

`noise sdlc bootstrap --target DIR` prepares an existing, unadopted repository
for agent-led planning without selecting or installing a lifecycle. It writes
three bounded artifacts:

- a byte-zero generated block in `AGENTS.md` that points the agent to the
  one-time instructions while preserving all existing prose;
- root `BOOTSTRAP.md`, which tells the agent to inspect authority, collect the
  consequential operator decisions, show the exact plan, and obtain approval
  before adoption; and
- `.sdlc/bootstrap.json`, a local receipt binding the generated block and
  document digests plus whether bootstrap originally created `AGENTS.md`.

The command does not inventory harnesses, read an answers file, contact Deck,
launch a provider, or authorize spend. An identical repeat is a read-only
success. A partial bootstrap, edited generated block, changed or missing
`BOOTSTRAP.md`, invalid receipt, symlinked managed path, unmanaged lifecycle
collision, or existing adoption is rejected without overwrite. Deleting the
visible instructions therefore does not cause silent reseeding while the
receipt remains.

Planning validates pending bootstrap state and includes explicit
`consume-bootstrap` actions. `adopt --yes` revalidates the same digests at the
write boundary, replaces the temporary `AGENTS.md` block, consumes the
unchanged bootstrap document and receipt, preserves pre-existing agent prose,
and transfers original `AGENTS.md` ownership into the adoption receipt so
later removal behaves correctly. The anchored transaction binds the inspected
file objects and bytes, atomically moves existing objects aside before it
verifies or replaces them, and writes `.sdlc/adoption.json` last as the commit
marker after bootstrap cleanup. Rollback restores only transaction-installed
objects; a concurrent user edit is preserved and the original is retained at
the reported transaction-backup path for explicit recovery.

This seed-once lifecycle is informed by OpenClaw's workspace bootstrap model,
where a new workspace receives a one-time `BOOTSTRAP.md` that is not recreated
after completion. Noise SDLC deliberately does not copy OpenClaw's broader
identity/workspace file set or runtime database: it adds a project-local
receipt and retains this project's exact-plan, explicit-approval, anchored
path, and integrity boundaries. See OpenClaw's official
[`bootstrapping` documentation](https://docs.openclaw.ai/start/bootstrapping)
and
[`workspace implementation`](https://github.com/openclaw/openclaw/blob/main/src/agents/workspace.ts).

## Inputs

For incubation, the lifecycle and execution IDs are fixed to
`noise/agent@0.1` and `deck/balanced@0.2`. Direct mode may spell them out;
guided mode recommends those same IDs. The four questions are:

| Answer | Values | Material consequence |
|---|---|---|
| `authorship` | `agent`, `mixed`, `human` | determines whether exact-head independent review is mandatory for all changes, agent-authored changes, or advisory until an agent authors work |
| `delivery` | `software`, `research`, `both` | selects deterministic/runtime evidence, research evidence, or both; research-only remains an explained unresolved fit during the one-seed experiment |
| `risk` | `ordinary`, `elevated` | makes high-capability implementation and explicit human approval mandatory for named high-risk work classes |
| `routingPreference` | `included-first`, `balanced`, `capability-first` | changes the ranking of eligible model candidates, never their capability or independence constraints |

The optional `highRiskClasses` list is required only for `risk: elevated` and
contains short public labels such as `migrations` or `release-signing`. It does
not weaken the seed's always-ask boundary for credentials, spend, deployment,
publication, or other external effects.

The answers file follows
[`answers.schema.json`](schemas/answers.schema.json). It may also declare public
harness/model IDs and an entitlement class when no safe probe can establish
them. `operator-declared` is provenance, not certification. Account identity,
credentials, balances, quotas, config paths, raw probe output, and a complete
machine inventory are forbidden.

The [example answers](../examples/sdlc-answers.json) describe an ordinary Go
software project with agent-authored changes and one project rule for build,
vet, and test checks. Copy the file and adapt its answers and check commands
to your project. Preview the result without writing project files:

```sh
noise-sdlc plan --target /path/to/project --answers /path/to/answers.json
```

The example leaves `operatorCapabilities` empty so it assumes no subscriptions
or model access. Roles without eligible inventory remain explicitly unresolved;
a successful plan does not mean those routes are ready to use. To declare
capabilities, use catalog harness and model IDs from `noise-sdlc inventory`,
specify your actual entitlement (`included`, `metered`, or `unknown`), and set
`allowUnverified` to `false` unless you explicitly accept an unverified route.
Declarations do not install a harness, authenticate, or authorize spend.

Direct and replay modes may also carry typed `projectRules`. These are not
wizard toggles: they preserve already-authoritative project-specific policy.
IDs must use `project.<slug>.*`; the same assurance and owner rules as the seed
apply; and local rules may not collide with or weaken seed rules.

## Allowlisted observations

The planner may inspect only:

- whether the target is a Git worktree and whether it has a commit;
- the existence and names, not arbitrary contents, of `AGENTS.md`, `SDLC.md`,
  `.sdlc/project.yaml`, `.sdlc/adoption.json`, supported language manifests,
  CI directories, and known task/review/visual-diff markers;
- the root `AGENTS.md` managed-block markers when present;
- the root `AGENTS.md` bytes as an opaque value only when preserving or
  replacing the managed byte-zero block;
- public Git metadata needed for a plan: root-relative tracked paths and HEAD;
- executable presence and version through a catalog descriptor's exact probe;
- sanitized public model selectors through a descriptor's exact enumeration
  probe (uncurated selectors are visible but non-routable);
  and
- operator-provided answers and capability declarations.

It may not read environment values, provider configuration, credential stores,
authentication output, repository file bodies outside its managed artifacts,
remote repository state, account state, balances, quota, or telemetry.

Every observation in a plan is labeled `observed`, `operator-declared`, or
`unknown`. Inference is never serialized as an observation.

## Recommendation rules

The planner produces exactly one recommendation and no more than two material
alternatives:

1. validate the fixed seed and execution template;
2. filter inventory candidates by installed and version-contract state;
3. exclude unavailable candidates and keep unverified/operator-declared ones
   visible but unresolved unless the operator explicitly allows them in the
   answers file;
4. satisfy hard role constraints before ranking preferences;
5. rank by the selected cost/capability preference, then template order, then
   stable harness/model ID lexical order;
6. enforce model-family independence for formal review against every bound
   implementation route;
7. bind fallbacks and two-round repair escalation; and
8. report any unsatisfied role without inventing availability or compatibility.

An alternative must differ in a consequential answer or binding and explain
that tradeoff. Individual policy rules are not exposed as toggles.

## Stable plan

The emitted document follows [`plan.schema.json`](schemas/plan.schema.json).
Canonical JSON is UTF-8, sorted by object key, uses two-space indentation, a
single trailing newline, and preserves array order only where the contract
defines preference or output order. Timestamps and machine paths are excluded.
For the same catalog bytes, seed bytes, normalized project facts, inventory,
and answers, the output bytes are identical.

The plan contains:

- input digests and public IDs;
- causal observations and answers;
- selected rules and unresolved fit;
- exact role bindings, fallbacks, independence constraints, and escalation;
- at most two alternatives;
- every target path, action, mode, content digest, and rollback action; and
- warnings that integrity is not conformance and routing is not execution or
  spend authorization.

`--answers FILE` and direct pinned flags must produce the same plan as the
interactive answers they represent. Unknown keys, missing consequential
answers, duplicate declarations, non-public fields, or a changed input digest
fail closed.

## Installation containment

Planning, adoption, update, removal, and rollback resolve every managed path
relative to the lexical project root and must not follow symbolic links. Before
staging or mutation, the implementation rejects a symlink at any existing path
component from the project root through the managed leaf, including `.sdlc`
itself. A managed leaf may be a missing path or a regular file only; managed
directories must be real directories. Checks alone are not a mutation boundary:
the transaction opens and retains a root-anchored directory capability, and
every create, rename, restore, and removal is performed relative to that
capability with beneath/no-escape semantics. A parent or leaf swap after
validation therefore fails rather than redirecting the operation outside the
opened project root. Failure leaves external paths untouched and causes
rollback to use the same retained root capability.

The deterministic suite covers symlinked parent directories, symlinked managed
leaves, and replacement of a validated component before commit. These cases
fail closed without modifying either the link target or unrelated project
files.
