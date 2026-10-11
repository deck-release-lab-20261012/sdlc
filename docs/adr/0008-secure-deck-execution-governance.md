# ADR 0008: Separate open routing from secure Deck execution governance

**Status:** Accepted product direction; implementation deferred
**Date:** 2026-08-14

## Context

Some orchestrators silently substitute older or otherwise unapproved models
when an intended route is unavailable. The current SDLC execution matrix is a
recommendation and cannot authorize or constrain a Deck launch. Browser
authentication is also the wrong boundary for execution governance: an agent
that can request a launch must not be able to relax the installation's policy.

The existing Deck synchronization contract exchanges runtime observations. It
does not certify interactive launches, authorize providers, or establish a
protected project execution grant.

## Decision

Deck has one durable installation-wide execution-governance mode, independent
of browser authentication:

- `open` preserves explicit, unconfined harness and model selection. SDLC
  recommendations remain advisory, and uncertified or role-mismatched choices
  are visibly unverified.
- `secure` requires every launch to match a current operator-approved project
  execution bundle and a protected Deck project receipt. Missing, stale,
  uncertified, unresolved, unavailable, or mismatched authority fails before
  Deck creates a session row, tmux pane, or provider process.

Mode is never selected per launch. Only an offline operator action under
Deck's backend lock may transition it. Entering or re-entering secure mode
increments an execution generation and invalidates every prior protected
project receipt. An agent cannot change mode, widen a binding, approve its own
bundle, or request an arbitrary harness or model as authority.

Secure mode has no implicit fallback, guessed equivalence, last-known-good
execution, older-model substitution, or automatic downgrade to open mode.
Only fallback-role edges explicitly approved in the project bundle are
eligible. A secure installation may exist before any route is certified; such
launches fail with a typed, content-free reason such as
`no-certified-interactive-route`.

## Operator confirmation

Approval and installation are separate operator actions:

1. `noise sdlc` constructs and strictly verifies a canonical candidate project
   execution bundle.
2. An operator reviews and confirms its exact digest through an SDLC-owned
   approval/export command.
3. The operator performs an authenticated or offline Deck import gesture that
   names the exact digest and stable Deck project ID.
4. Deck independently verifies the bundle and stores a protected receipt that
   binds the project ID, bundle digest, current execution generation, and
   import/confirmation evidence.

Deck inventory may satisfy current availability but cannot manufacture
approval or alter bundle content. A cryptographic signature is optional for
the first same-host implementation when canonical bytes plus the protected
receipt provide equivalent integrity. The bundle envelope remains
signature-ready for later cross-host use.

## Ownership

`noise sdlc` owns the catalog, compatibility certification, lifecycle-role
taxonomy, execution templates, binding resolution, fallback graph,
model-family independence rules, canonical bundle export, operator
confirmation, and strict verification. It does not launch providers,
authenticate, authorize spend, or treat inventory as certification.

Deck owns fresh runtime inventory, provider adapters, the durable mode and
execution generation, protected project receipts, bundle import, pre-launch
enforcement, session creation, and content-free rejection audit. Deck does not
recreate the SDLC catalog or role policy.

## Orchestrator role

`orchestrator` is a first-class lifecycle role; it is not an alias for
`implementation.high` or `repair`. It owns decomposition, assignment,
monitoring, integration coordination, and gate enforcement. It may perform
read-only inspection and controlled integration operations, but does not
author substantive production changes.

Its binding requires an approved high-capability model, a declared reasoning
floor, an explicit fallback chain, and the same exact harness/model/version
authority as every secure role. It may launch only roles and fallback edges in
the approved bundle. If it makes a substantive repository change, that action
is implementation work and requires the normal independent formal review.
Formal review remains model-family-independent from the applicable
implementer; bundle eligibility does not itself approve a Git change.

## Interactive certification boundary

Existing harness contracts cover noninteractive SDLC commands. They do not
certify the interactive sessions Deck launches. Secure-mode routability
requires a distinct immutable certification for the exact harness, executable
version, model ID, reasoning level, and Deck interactive launch contract. The
evidence must include separately authorized live execution and Deck
interoperability results and bind their digests. Until at least one exact tuple
passes, every secure Deck launch remains unavailable.

Unconfined capacity-routing experiments remain disabled by default and use
open mode; they do not require secure-mode certification.

## Deferred implementation

This ADR changes design authority only. It does not add a bundle schema,
approval command, verifier, certification claim, Deck mode transition, receipt
import, or launch gate. Those changes require a separately approved stack and
must not be inferred from the current advisory execution matrix.

The follow-on stack is ordered as:

1. canonical bundle, approval-envelope, interactive-certification, and strict
   verification-result schemas;
2. lifecycle taxonomy and template support for `orchestrator` plus acyclic
   fallback validation;
3. SDLC approval/export and independent verifier implementation;
4. exact interactive certification fixtures and authorized live evidence; and
5. Deck durable mode, generation-bound receipt import, and central pre-launch
   enforcement in the Deck repository.

The follow-on design still requires operator approval of the canonical format
and digest algorithm, the same-host confirmation-evidence representation, the
exact orchestrator model quality and reasoning floor, Deck receipt protection
and recovery mechanics, and the closed content-free rejection taxonomy. Until
those decisions and schemas are accepted, implementations must not invent
defaults or claim secure-mode compatibility.
