# Invocation contracts and live guidance

An adopted route identifies a harness, model, family and reasoning level. That
selection alone does not explain prompt transport, process completion or review
publication. Catalog `noise/harnesses@0.5` adds optional invocation schema version
1 to individual harness version ranges. Missing actions mean unsupported, not an
invitation to guess flags or silently substitute a model.

## Find the adopted contract

```sh
noise sdlc execution show --target .
noise sdlc execution show --target . --role review.formal --json
noise sdlc execution show --target . --role implementation.high --harness-version 2.1.263
```

`show` verifies a bounded snapshot of the managed project files, uses the adopted
roles and compares the receipt's catalog digest with the available catalog. It
does not discover or launch harnesses. Catalog mismatch returns an explicit state
and an update-plan next step without presenting replacement contracts. Broken
adoption returns a diagnosis next step; generic `noise sdlc help execution`
remains available. No project files are changed.

Without a version, guidance lists declared ranges for the selected route. A
supplied version filters those ranges but remains a caller declaration. The
runner must establish the installed version and current authority independently.
Experimental descriptors are not live certification. Initial action coverage is
Claude 2.1.263 and OMP 18.1.11; older ranges retain their original declarations.
Codex routes remain explicitly unsupported for invocation actions in this
release. An existing adoption retains its old catalog digest until an explicit
update is applied.

Generated SDLC guidance adds a short pointer to this command. AGENTS.md remains
an orientation and authority entry point. Further reductions to static policy
must preserve the discoverable safety core and go through generated sources and
the adoption/update planner. Live help supplies detail within the Tasks/Deck
workflow: Tasks owns acquisition and work state; Deck owns sessions and capacity.

## Descriptor and runner responsibilities

Each action declares an exact argument vector, working-directory transport,
prompt transport and byte bound, permission limitations, output normalizer and
bound, and completion conditions. Review has a separate publication contract.
The committed JSON schema and semantic validator reject unknown placeholders,
inconsistent transport, absent review readback and unsupported contract versions.

Claude uses UTF-8 stdin and process cwd. OMP uses a final `@{promptFile}` argument
and an explicit cwd argument. Pure preparation substitutes typed values as argv
elements, never as shell text. It preserves prompt bytes and returns a declaration
for the runner; SDLC creates no prompt file and starts no provider. For an OMP
prompt file, the runner owns secure exclusive creation, boundary enforcement and
cleanup of that exact artifact. It must not overwrite an existing path or follow
symlinks merely because a lexical preparation plan was accepted.

Both initial actions request final text. Native event streams and hidden reasoning
are not inputs to this normalizer. Process outcome and output normalization are
separate: valid output cannot erase a failed process. Missing output, invalid
UTF-8 or an exceeded bound make normalization invalid. A successful process with
valid final output still does not establish task completion.

For review, the final nonempty line must carry the `noise review` audit object.
Parsing identifies a claimed review; it does not prove that a bot published it,
that its commit matches the candidate, that coverage was complete, or that the
reviewer was independent. Review owns publication and authoritative readback.
An uncertain result must be reconciled before any retry, because a verdict may
have posted before local output handling failed. Model-family and exact-head
policy comparisons are separate integration obligations.

Permission flags describe harness behavior, not a proven read-only boundary.
The external runner must enforce the required isolation and current authorization.
SDLC preparation and help grant neither execution nor publication authority.

## Evidence and limits

The initial syntax comes from bounded public `--version` and `--help` observations
of installed Claude and OMP. OMP's public print-mode declaration specifies final
response text versus JSON events. The review handoff uses a final-line audit
object followed by authoritative publication readback, as described in the
[review evidence contract](review-evidence-contract.md).
These observations do not certify successful provider execution, credentials,
spend, confinement, or a complete review round. Hermetic tests cover descriptors,
argument and prompt preservation, normalization failures, stale adoption,
unsupported routes and help without provider launches.
