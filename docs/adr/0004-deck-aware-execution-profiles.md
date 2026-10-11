# ADR 0004: Catalog tested harness contracts and compile execution profiles

**Status:** Accepted; Deck sync details extended by ADR 0006
**Date:** 2026-08-13

## Context

Agent-built projects need more than lifecycle gates. An operator repeatedly
chooses which installed CLI and model should perform cheap bulk implementation,
ordinary work, difficult repair, and independent formal review. The choice also
depends on subscription versus metered access, model-family independence,
reasoning controls, CLI argument syntax, and escalation after failed rounds.

That policy is currently repeated manually. Deck can execute provider CLIs,
but there is no accepted general Deck compatibility API for another tool to
consume. `noise sdlc` needs enough Deck awareness to make the execution policy
portable without becoming the runtime, credential store, or provisioner.

## Candidate decision

During incubation, `noise sdlc` will own four related artifacts:

1. A versioned curated registry of supported harness descriptors and model
   capabilities, loaded from independently updateable fragments.
2. A private, disposable local inventory produced only by descriptor-approved
   read-only probes.
3. Curated portable execution templates describing lifecycle roles, ranked
   preferences, cost/quality constraints, independence, fallbacks, and
   escalation.
4. An operator-confirmed execution binding rendered into the effective SDLC.

The catalog knows exact argv construction and safe discovery behavior for each
supported CLI/version range. The resolver may recommend exact harness/model
bindings from the local inventory, but adoption shows the complete matrix and
requires confirmation. Unknown capability stays unknown.

The committed binding may name public harness/model identifiers and routing
rules. It must not contain account identity, credential location, token,
balance, quota, raw auth output, or a complete local inventory. The disposable
inventory is uncommitted by default and is not included in the adoption
receipt.

`noise sdlc` does not install, configure, authenticate, launch, resume, or
authorize spend for a harness. Deck remains the intended runtime executor.
ADR 0006 defines the stable public manifest/receipt contract. Deck integration
remains incomplete until Deck implements its producer endpoint; it must not be
inferred from Deck internals.

## Support contract

A harness/version is supported only when its applicable contract suite passes:

- schema and normalized-capability validation;
- exact argv-vector construction for model, reasoning, noninteractive, and
  structured-output controls;
- hermetic fake-CLI coverage of discovery, malformed output, prompts,
  timeouts, sanitization, and failures;
- deterministic execution-profile resolution tests;
- opt-in live discovery/model-selection tests against the real CLI; and
- when available, a Deck import/launch interoperability test.

Tests capable of consuming quota or money are never ordinary CI. They require
an explicitly authorized test identity and budget. Certification records the
tested CLI versions and date. A newly detected version outside that evidence is
`unverified`, not silently assumed compatible.

## Rationale

This makes the project more than a governance template while preserving lane
ownership. The SDLC can state both the work policy and the chosen execution
strategy; Deck can execute it; review and other tools can judge their own
evidence. The operator no longer needs to restate a model-routing matrix for
every project, and a CLI argument change becomes a detectable contract break.

Separating catalog, inventory, template, and binding prevents volatile local
availability from contaminating durable policy. It also permits a useful
public catalog without publishing anyone's subscriptions or credentials.

## Consequences

- Harness maintenance and live compatibility testing become real product work.
- Model/pricing freshness must be explicit; recommendations need catalog and
  observation timestamps.
- The resolver must explain selections and unresolved constraints rather than
  emit an opaque score.
- Subscription access is treated as a payment/entitlement mode, not zero cost.
- Exact model choices can be pinned while a portable template preserves their
  intent and enables later re-resolution.
- The shared descriptor and receipt contract is versioned independently of
  Deck's fast-moving implementation. This project must not copy private Deck
  internals or claim launch compatibility before interoperability evidence.

## Rejected alternatives

### Keep model routing only in prose

This reproduces the current manual process and cannot determine whether the
named harness/model is locally available or whether its invocation syntax has
drifted.

### Let Deck own all catalog and policy data

That makes SDLC adoption unable to explain or resolve execution policy without
a running Deck integration, and it couples durable lifecycle intent to one
runtime. Deck should execute the binding, not be its only source of meaning.

### Probe provider configuration and credentials directly

Direct inspection creates privacy and security risk, depends on unstable
internal files, and turns an SDLC tool into an auth auditor. Official bounded
probes plus explicit operator declaration are sufficient and honest.

### Treat any installed executable as supported

Presence proves neither compatible arguments nor usable models. Supported
versions require contract evidence; unknown versions remain visible but
unverified.
