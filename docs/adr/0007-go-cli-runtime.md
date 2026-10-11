# ADR 0007: Use Go for the standalone CLI runtime

**Status:** Accepted

**Date:** 2026-08-14

## Context

The incubation implementation began in Python to test the product boundary.
It now has a real standalone command surface, safe subprocess probing, a
loopback web UI, deterministic rendering, and optional Deck synchronization.
Keeping that runtime as an uninstalled Python source tree makes the tool harder
to distribute and does not match the intended shape of a small Noise CLI.

## Decision

The authoritative `noise-sdlc` runtime is Go. Domain data is represented with
typed structures; untyped JSON is limited to forward-compatible Deck receipt
boundaries and rendered output documents. The repository launcher prefers a
prebuilt binary and otherwise uses the local Go toolchain for incubation.

The command and JSON contracts remain compatible with the Python prototype.
The prototype may be removed only after byte-level plan parity and lifecycle,
inventory, Deck, and web tests pass.

## Consequences

The tool can become a single binary without requiring a Python environment.
The migration adds Go-native tests and makes Go the implementation language,
but does not authorize publishing binaries or packages. Deck integration
remains optional and standalone inventory remains best-effort.
