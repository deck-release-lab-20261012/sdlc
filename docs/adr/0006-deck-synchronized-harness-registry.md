# ADR 0006: Synchronize an updateable harness registry with Deck

**Status:** Accepted incubation direction; Deck producer pending
**Date:** 2026-08-13

## Context

The first catalog encoded five locally known harnesses in one file. That caught
argument errors, but it made discovery depend on the catalog maintainer already
knowing every harness. Harnesses, model selectors and Deck's adapter set can
change independently. A manually copied routing table cannot remain current.

Deck currently exposes its console and component version but no public harness
manifest endpoint. Copying its private database or configuration would couple
the tools, risk credential exposure, and still fail when internal shapes move.

## Decision

Keep standalone operation as the default and use a dual-authority,
content-addressed synchronization contract only after explicit opt-in.

- Standalone discovery is explicitly `standalone-best-effort`: safe executable,
  version, and allowlisted model probes plus operator declarations. It does not
  claim runtime readiness, quota, or spend authority.
- After the operator opts in, Deck owns runtime adapter presence, installed
  versions, safely enumerable public model IDs, and observation time.
- `noise sdlc` owns compatibility evidence, normalized policy metadata,
  execution templates, and operator-confirmed bindings.
- Harness descriptors are sorted drop-in JSON fragments. A new harness changes
  registry data and contract tests, not resolver code.
- `noise sdlc` exports a deterministic bundle with the full public descriptor
  set, templates, interaction contract, and catalog digest.
- Deck returns a normalized receipt naming the digest it accepted. Valid
  receipts are stored atomically as last-known-good state.
- A mismatched digest, unknown harness, or version outside a certified range is
  visible as drift, uncataloged, or unverified. None is silently routable.
- After opt-in, Deck change events are the preferred refresh trigger. Refresh
  at Deck startup and a bounded loopback poll are fallbacks. Standalone mode
  performs no background Deck contact.

The same interaction contract drives the terminal and the web UI. The
incubation web server binds only to loopback, caps JSON bodies, exposes no
credential data, and requires an ephemeral action token plus the exact plan
digest before adoption. Deck may embed or reverse-proxy this surface after it
implements the producer contract.

Deck exposes reusable runtime inventory as `noise deck harnesses
list|show|refresh|status|open`. The Harnesses web surface owns the optional
connection control. No `noise deck sdlc` command family is accepted during
incubation. This repository does not patch the installed Deck executable or
create a second Deck configuration surface.

When Deck adds a project, it may offer the shared selection workflow as an
optional post-creation card. Skip is the default, dismissal is project-scoped,
and persistent synchronization is a separate consent. The SDLC workflow cannot
block or roll back adding the project.
A verified existing adoption suppresses the offer; invalid managed artifacts
surface as repair/drift rather than a fresh onboarding opportunity.

## Freshness state machine

```text
Deck adapter change
  -> publish normalized manifest
  -> schema and size validation
  -> atomic last-known-good receipt
  -> compare accepted catalog digest
     -> equal: in-sync
     -> different: catalog-drift
  -> surface unknown harnesses/versions
  -> test and curate descriptor update
  -> Deck accepts new digest
```

Fresh observation and support certification deliberately move at different
speeds. This is how Deck can update daily without making an untested CLI
silently eligible for delivery work.

## Consequences

- The Deck repository needs one bounded producer endpoint or event containing
  the public receipt schema. This repository does not infer it from private
  Deck internals.
- Registry updates can be proposed automatically from observed drift, but
  promotion to supported still requires contract evidence.
- Offline planning uses the last-known-good receipt and displays its age.
- The protocol can later travel over a local socket or authenticated remote
  transport without changing authority semantics; incubation permits loopback
  HTTP and file exchange only.

## Rejected alternatives

### Mirror Deck's database

It creates an unstable private dependency and risks copying auth/config state.

### Treat Deck inventory as support evidence

Presence proves freshness, not argv compatibility or safe failure behavior.

### Update only on `noise sdlc` releases

That leaves new Deck adapters invisible and recreates the operator's manual
routing burden.
