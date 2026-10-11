# Deck synchronization contract

Deck integration is optional. Without it, the tool uses local safe probes and
marks inventory `standalone-best-effort`; readiness and spend remain unknown.
Live subscription quota is never among them: it is a per-account runtime fact
owned by whatever holds the provider session, and this tool does not report it
in any state, synchronized or standalone. Running `sync accept`, `sync pull`,
or enabling the integration from Deck is the opt-in action. Exporting the
public bundle is read-only and does not enable synchronization.

`noise sdlc sync export` emits a deterministic `noise.sdlc.deck-sync` bundle.
It contains the public harness registry, execution templates, shared
interaction contract, authority declaration, and canonical catalog digest. It
contains no account, credential, balance, quota, raw authentication, or
provider configuration data.

Deck answers with `noise.deck.sdlc-receipt`:

```json
{
  "schemaVersion": 1,
  "kind": "noise.deck.sdlc-receipt",
  "catalogDigest": "sha256:<64 lowercase hex characters>",
  "deckVersion": "<public build or release>",
  "observedAt": "2026-08-13T15:30:00Z",
  "inventory": {
    "schemaVersion": 1,
    "source": "deck-runtime",
    "catalog": "noise/harnesses@0.4",
    "harnesses": []
  }
}
```

The inventory uses the normalized inventory contract, but Deck supplies
observation-only values. Harness state is limited to `absent`, `installed`,
`ready` or `enumerated`; `rangeStatus` is `unverified` or `absent`. `unknown`
labels are limited to `installed`, `ready`, `models`, `version`, and
`spend-authority`, so a receipt cannot introduce a fact this tool does not own
and live subscription quota is never a valid label. A model contains only its
public selector: provenance is `enumerated`, family is null,
quality and entitlement are `unknown`, reasoning levels are empty, and
`verified` is false. Deck may report uncataloged harness IDs so they become
visible in sync status. After receipt validation, `noise sdlc`
recomputes range and model certification from the digest-matched local catalog;
Deck cannot assert those locally owned conclusions. Nothing is routed until a
tested descriptor and model policy exist.

Receipt transport during incubation is one of:

- file exchange through `noise sdlc sync accept RECEIPT`;
- bounded loopback pull through `noise sdlc sync pull URL`; or
- loopback POST to `/api/deck-receipt` using the ephemeral action token from
  `/api/bootstrap`.

## Deck command surface

Deck owns native harness inventory and SDLC opt-in under its existing Noise
noun:

```text
noise deck harnesses list             # normalized runtime inventory
noise deck harnesses show <HARNESS>   # one adapter's public facts
noise deck harnesses refresh          # bounded adapter/model refresh
noise deck harnesses status           # freshness, adapter errors, and sync state
noise deck harnesses open             # open the Harnesses surface in Deck
```

The Harnesses web surface owns an explicit **Connect noise sdlc** control. It
must display the endpoint, public fields exchanged, and initial catalog digest
before confirmation. Disconnecting stops sync and discards only Deck-owned
connection metadata. Neither action alters an SDLC adoption, authorizes spend,
installs harnesses, or sends credentials. Disconnecting leaves `noise sdlc`,
its standalone best-effort inventory, and project artifacts fully usable. Deck
change events and startup refresh run only while connected.
`harnesses refresh` remains useful without `noise sdlc`; when integration is
enabled, a successful inventory change also emits a new SDLC receipt.

No `noise deck sdlc` command family is committed. A CLI connect/disconnect
operation should be added only if Deck canary use shows that the web control is
insufficient.

## New-project offer

After Deck has successfully added a project, it may show a non-modal card:
**Choose how agents ship this project**. The card offers **Start selection** and
**Skip**, defaults to Skip, and never blocks or rolls back project creation.
Dismissal is remembered for that project rather than becoming a global
preference.

Deck evaluates eligibility before showing the card. A project with an adoption
that passes `noise sdlc verify` is already onboarded, so the card is suppressed.
A project-level dismissal also suppresses it. If managed SDLC artifacts exist
but verification fails, Deck shows a drift/repair state instead of treating the
project as new or offering selection again.

Deck consumes the same read-only classifier available through `noise sdlc
onboarding status --target PROJECT` and `GET /api/onboarding`. It returns one of:

- `offer-selection` / `no-adoption`;
- `already-onboarded` / `verified-adoption`; or
- `repair-required` with a normalized reason such as
  `managed-output-drift`, `missing-authority-input`, or
  `unmanaged-sdlc-collision`.

The classifier never writes. Project-scoped dismissal remains Deck-owned and is
applied after this eligibility result.

Starting the workflow opens the shared guided planner for that project. It is
an opt-in to one selection session, not consent to persistent Deck sync.
**Keep routing synchronized with Deck** is a separate, unchecked choice with
its own disclosure and confirmation. Closing or skipping the workflow writes
no SDLC artifacts and enables no background activity.

The installed Deck component at the time of this decision supports `serve`,
`version`, `backup`, and `restore`; `harnesses` and its web surface are therefore
Deck-side additions, not aliases implemented by this repository.

The receiver validates schema, size, timestamp shape, and normalized inventory
before atomically replacing `.sdlc/deck-sync.json`. Invalid input leaves the
previous receipt untouched. Equal catalog digests mean `in-sync`; unequal
digests mean `catalog-drift`. Offline use reads the last-known-good receipt and
must display its observation time.

`noise sdlc harness candidates` turns the last receipt's unknown harnesses,
models, and versions into a compact curation queue. Candidate output is not a
support claim; it tells the maintainer which descriptor metadata and contract
tests are still required.

## Open and secure execution governance

[ADR 0008](adr/0008-secure-deck-execution-governance.md) defines a separate,
durable Deck execution-governance mode. The current synchronization receipt and
advisory execution matrix support `open` mode only. They are not a secure-mode
authorization and must never be accepted as one.

The follow-on SDLC stack will define four canonical artifacts:

- `project-execution-bundle.schema.json`: the content-addressed, operator-
  confirmed role and launch authorization;
- `execution-bundle-approval.schema.json`: confirmation evidence for the exact
  canonical bundle digest, with a signature-ready envelope;
- `interactive-certification.schema.json`: immutable live and Deck
  interoperability evidence for one exact harness/version/model/reasoning
  tuple; and
- `execution-bundle-verification.schema.json`: a content-free strict verifier
  result suitable for Deck import and audit.

The project bundle must bind its schema and format versions; catalog,
execution-template, effective-policy, and adoption digests; every required
lifecycle role including `orchestrator`; exact harness, model, reasoning floor,
supported executable versions, and interactive-certification digests; explicit
acyclic fallback-role edges; unresolved-role state; and reviewer model-family
independence constraints. It contains no credentials, quota data, provider
output, prompts, workspace paths, launch tokens, or machine inventory.

Deck's protected import receipt is Deck-owned and binds a stable Deck project
ID, the exact approved bundle digest, the current installation execution
generation, and import/confirmation evidence. In secure mode, absence or
invalidity fails before any launch artifact is created. The exact Deck receipt
schema and rejection taxonomy are part of the separately approved Deck
handoff; this repository does not pre-implement them.
