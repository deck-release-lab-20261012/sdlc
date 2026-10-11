# noise sdlc

A Go CLI that installs a curated, versioned software-delivery lifecycle into a
project — policy, agent-harness routing, and verification — as byte-exact
managed files whose integrity it can check later.

**Status: experimental.** Daily-driven by its author; interfaces may change
between releases. The first public release is v0.3.0. See
[DEVELOPMENT.md](DEVELOPMENT.md) for the development model (curated release
mirror, pull requests disabled, issues open) and [SECURITY.md](SECURITY.md)
for vulnerability reporting.

Part of the Noisefloor suite — infrastructure for running fleets of
third-party coding agents against real repositories. Each tool works
standalone; none requires the others.

## What it does

Projects that run coding agents need lifecycle policy: which rules govern
changes, which harnesses and models may fill implementation and review roles,
and how drift in those decisions is detected. Copying another project's
governance files rots quickly. noise-sdlc instead ships a tested policy seed
(`noise/agent@0.1`), a harness catalog (`noise/harnesses@0.5`), and an
execution template (`deck/balanced@0.2`), and installs them into your project
as managed files whose exact bytes are recorded.

The default path is a short, opinionated interview (or a plain answers JSON
file) that produces one explained plan. `adopt --yes` installs exactly the
planned bytes: a managed `AGENTS.md` block, `SDLC.md`,
`.sdlc/project.yaml`, and `.sdlc/adoption.json`. `verify` re-checks those
bytes at any time; `diagnose` explains artifact integrity separately from
compatibility with neighboring tools; `remove` uninstalls.

The harness side is a closed catalog of tested CLI descriptors. `inventory`
probes installed harnesses using only the cataloged version and
model-enumeration commands (five- and ten-second timeouts). `execution plan`
resolves bulk, standard, high-risk, formal-review, and repair routes against
the capabilities you declare, explains each choice, and enforces reviewer
model-family independence. Declaring a capability is your statement about
what you have installed and are entitled to use; an empty set is valid and
leaves routes unresolved rather than inventing ones you cannot run.

## What it does not do

- **It does not start harness sessions or call models.** Execution bindings
  are routing policy only; they do not authenticate or authorize spend.
  `inventory`, `plan`, `adopt`, `execution plan`, and `web` (at startup)
  execute installed harness binaries for the catalog's version and model-list
  probes, including discovery-candidate version probes, with five- or
  ten-second timeouts. `plan`, `adopt`, `execution plan`, and `web` (on each
  plan/adoption request) run `git rev-parse HEAD` in targets containing `.git`
  for a read-only revision lookup, with no explicit timeout. `diagnose` runs
  `noise tasks version`, `noise tasks context --json`, and
  `noise tasks verify --json` with five-second timeouts. All run in the target
  project; those binaries may perform their own filesystem or network I/O.
- **It handles no credentials.** It reads no credential configuration,
  environment secrets, or authentication output, and exchanges no account,
  quota, or balance data in any state, synchronized or standalone.
- **It is not a scheduler, sandbox, or review authority.** It does not run
  your pipeline, enforce permissions at execution time, or publish review
  verdicts. `review request`/`review compare` derive expectations and compare
  declared evidence; a match is neither authenticated approval nor merge
  readiness (see the
  [review evidence contract](docs/review-evidence-contract.md)).
- **Integrity is not conformance.** A passing `verify` means the managed
  files match their recorded digests. It is not lifecycle conformance, review
  approval, security evidence, runtime acceptance, or merge readiness.
- **The tool itself opens network connections only on loopback, on request.**
  `web` listens only on loopback and `sync pull` connects only to loopback
  endpoints. Schema validation resolves references
  within the loaded document and the bundled metaschemas; it never fetches
  from the network or filesystem.

## Install

Requirements: Linux on x86-64 (the only platform this experimental release is
qualified on) and Go 1.26.9 or newer to build from source.

```sh
git clone https://github.com/noisefloor-oss/sdlc.git
cd sdlc
git checkout v0.3.0        # or the latest release tag
go build -o noise-sdlc ./cmd/noise-sdlc
./noise-sdlc --version     # verify the build runs
```

Or install the command directly with Go:

```sh
go install github.com/noisefloor-oss/sdlc/cmd/noise-sdlc@v0.3.0
```

Move the binary anywhere on your `PATH`, for example
`install -m 0755 noise-sdlc ~/.local/bin/`. Running
`go install ./cmd/noise-sdlc` from the clone works too. The binary embeds the
catalog, policy seed, execution templates, and validation schemas, so it runs
from any directory — no data files need to be installed alongside it.

Release archives are published on GitHub Releases with SHA-256 checksums.
Verify downloads against the checksums file attached to the release.

## Quick start

The CLI's own writes below stay inside the scratch project, and only
`adopt --yes` writes managed files. `plan` and `adopt` also execute installed
harness binaries on your PATH for the catalog's bounded version and model-list
probes; those binaries may perform their own filesystem or network I/O.

```sh
# Validate the embedded catalog and policy seed (read-only)
noise-sdlc check-catalog

# Create a scratch project and a minimal answers file
mkdir -p /tmp/sdlc-demo && cd /tmp/sdlc-demo
cat > answers.json <<'EOF'
{
  "schemaVersion": 1,
  "seed": "noise/agent@0.1",
  "executionTemplate": "deck/balanced@0.2",
  "answers": {
    "authorship": "agent",
    "delivery": "software",
    "risk": "ordinary",
    "routingPreference": "balanced",
    "highRiskClasses": []
  },
  "operatorCapabilities": [],
  "projectRules": []
}
EOF

# Print the full adoption plan; writes nothing
noise-sdlc plan --target . --answers answers.json

# Install the managed lifecycle files
noise-sdlc adopt --target . --answers answers.json --yes

# Re-check the installed bytes at any time
noise-sdlc verify --target .

# Sync state: standalone and disabled until you opt into Deck integration
noise-sdlc sync status --target .
```

`operatorCapabilities` is where you declare the harness/model pairs you
actually have, for example
`{"harness": "omp", "model": "zai/glm-5.3", "entitlement": "included", "allowUnverified": true}`
(a harness/model pair from `noise-sdlc harness show omp`).
The answers format is defined by
[docs/schemas/answers.schema.json](docs/schemas/answers.schema.json). Without
`--answers`, `plan` and `adopt` ask the interview questions on stdin.

The command surface, grouped as in `noise-sdlc help`:

- **Discovery:** `list`, `show`, `harness`, `inventory`, `execution`,
  `review`
- **Adoption:** `bootstrap`, `plan`, `adopt`, `remove`, `onboarding`, `web`
- **Verification:** `diagnose`, `verify` (alias `status`), `check-catalog`,
  `check-evidence`
- **Deck synchronization:** `sync`

`bootstrap` is the agent-first entrypoint for a new repository: it adds a
small pointer block to `AGENTS.md` plus a one-time `BOOTSTRAP.md` that a later
`adopt` consumes. `web` serves the same guided adoption as a loopback browser
UI. Run `noise-sdlc help <command>` for per-command flags, exactly which
files each command reads and writes, and examples.

## Optional: dispatcher integration (noise / Noisefloor Deck)

noise-sdlc is fully usable standalone. Integration with the `noise`
dispatcher and the Noisefloor Deck desktop app is opt-in in both directions:

- `noise-sdlc sync export` emits a deterministic, credential-free bundle of
  the public harness registry, execution templates, and interaction contract
  that Deck can consume.
- Deck answers with a runtime receipt, delivered through
  `noise-sdlc sync accept RECEIPT`, `noise-sdlc sync pull URL` (loopback
  only), or the web UI. The receipt is observation-only; this tool recomputes
  all routing conclusions from its own catalog. `sync status` reports
  `in-sync` or `catalog-drift` from the recorded digests.
- `noise-sdlc harness candidates` turns harnesses Deck has seen but this
  catalog does not know into a curation queue. It is not a support claim.

Disconnecting or never connecting leaves every standalone feature intact.
The full exchange format is the
[Deck synchronization contract](docs/deck-sync-contract.md).

## Drift and repair limits

- `verify` checks the exact bytes of the managed `AGENTS.md` block,
  `SDLC.md`, `.sdlc/project.yaml`, and `.sdlc/adoption.json`, and scans the
  project tree for shadow lifecycle files. Any drift exits nonzero and names
  the file. Its scope is those generated artifacts only — see
  *What it does not do* above.
- `remove --yes` first requires the adoption to verify intact; it refuses to
  delete drifted files, because they may carry your edits.
- Re-running `plan` and `adopt --yes` regenerates the managed files from
  current answers and recorded inputs, which restores integrity after drift.
  Nothing repairs automatically; repair is always a deliberate operator
  action that overwrites the managed files with freshly planned bytes.
- `adopt` refuses to replace files it did not generate, such as a hand-written
  root `SDLC.md`; `bootstrap` fails closed on edited, partial, or conflicting
  bootstrap state. Repeating `bootstrap` is idempotent only while its
  generated artifacts remain intact.
- Adoptions pin the catalog and template they installed by digest. Upgrading
  the `noise-sdlc` binary never rewrites an existing adoption; a newer binary
  reports a stale catalog in `execution show` explicitly instead of
  substituting one.

## Uninstall and data retention

Everything noise-sdlc writes lives inside the target project:

- `adopt`: the managed `AGENTS.md` block (or the whole file, if it created
  it), `SDLC.md`, `.sdlc/project.yaml`, `.sdlc/adoption.json`
- `bootstrap`: the `AGENTS.md` pointer block, `BOOTSTRAP.md`,
  `.sdlc/bootstrap.json` (consumed by a successful `adopt`)
- `sync accept` / `sync pull` / the web UI: `.sdlc/deck-sync.json`

To remove an adoption, run `noise-sdlc remove --target PROJECT --yes`, which
deletes the managed files and reports the removal as unrecoverable. The
`.sdlc/deck-sync.json` receipt is left in place; delete it by hand if you
also want sync state gone. To uninstall the tool, remove the adoption from
each project as above, then delete the binary and the source clone. There is
no global state: no configuration directories, no accounts, no telemetry, and
nothing written outside the target project.

## Compatibility

This is an experimental release: interfaces, file formats, and behavior may
change between releases without a deprecation cycle. Within that caveat, the
documented surface is:

- **CLI:** the commands and flags printed by `noise-sdlc help` and
  `noise-sdlc help <command>`, including each command's stated reads and
  writes.
- **Exit codes:** 0 on success, 2 when a command rejects its input or fails,
  as documented in the help text. Note that `diagnose` exits 0 whenever it
  produced a report, including when the report requires action.
- **Structured output:** schema coverage is listed below; not every JSON
  output has a schema or a schema version. Progress and prompts go to stderr.
- **Versioned inputs:** the policy seed, harness catalog, and execution
  templates are content-versioned (`noise/agent@0.1`, `noise/harnesses@0.5`,
  `deck/balanced@0.2`); adoptions record the exact digests they installed.
- **Managed file layout:** `SDLC.md`, the managed `AGENTS.md` block, and the
  `.sdlc/` files above, verified by digest.

These JSON outputs carry `schemaVersion: 1` and conform to the linked schemas:

| Command output | Schema |
| --- | --- |
| `show` | [seed.schema.json](docs/schemas/seed.schema.json) |
| `plan`, `adopt` (the plan printed to stdout) | [plan.schema.json](docs/schemas/plan.schema.json) |
| `inventory` | [inventory.schema.json](docs/schemas/inventory.schema.json) |
| `diagnose` | [diagnosis.schema.json](docs/schemas/diagnosis.schema.json) |

Some outputs contain schema-covered objects without a schema for the whole
response. `review request` carries `schemaVersion: 1`; its `expectation` object
also carries version 1 and conforms to `$defs/expectation` in
[review-evidence.schema.json](docs/schemas/review-evidence.schema.json).
That schema's root describes the **input** to `review compare`, not either
command's complete output. `sync export` carries `schemaVersion: 1`; its
`catalog` and each `executionTemplates` entry carry version 1 and conform to
[harness-catalog.schema.json](docs/schemas/harness-catalog.schema.json) and
[execution-template.schema.json](docs/schemas/execution-template.schema.json),
respectively. `harness show` emits an unversioned descriptor covered by
`$defs/harness` in the catalog schema.

Receipt schemas describe files, not command summaries:
[bootstrap-receipt.schema.json](docs/schemas/bootstrap-receipt.schema.json)
describes `.sdlc/bootstrap.json`, and
[deck-sync-receipt.schema.json](docs/schemas/deck-sync-receipt.schema.json)
describes the receipt accepted by `sync accept`/`sync pull` and saved as
`.sdlc/deck-sync.json`; both receipts carry `schemaVersion: 1`.
Similarly, [evidence.schema.json](docs/schemas/evidence.schema.json) and
[evidence-v1.schema.json](docs/schemas/evidence-v1.schema.json) describe version
2 and legacy version 1 inputs to `check-evidence`, not its validation summary.

Other command outputs are informational and may change. `--version --json`,
`bootstrap`, `onboarding status`, `execution show --json`, `harness interaction`,
`harness candidates`, `review compare`, and `check-evidence` carry
`schemaVersion: 1` but have no standalone output schemas. Neither do the
complete `review request` and `sync export` responses described above.
`list`, `harness list`, `execution plan`, `check-catalog`, `verify`/`status`,
`sync status`, `sync accept`, `sync pull`, and `remove --yes` have neither
output schemas nor top-level schema versions.

Only the latest tagged release is supported; there are no backports. Linux
x86-64 is the only qualified platform for this release.

## Building and testing from a source checkout

```sh
go build ./...
go vet ./...
go test ./...
```

The binary normally uses its embedded resources. To test catalog or schema
edits without rebuilding, point `NOISE_SDLC_SOURCE_ROOT` at a source
checkout, for example `NOISE_SDLC_SOURCE_ROOT="$PWD" noise-sdlc check-catalog`;
missing or invalid files there are errors, never a silent fallback. The
repository-local launcher `./bin/noise-sdlc` rebuilds from current source
when its freshness receipt does not match, so it always runs what is checked
out.

## License

MIT © 2026 Doug Doan. Built at Noisefloor. See [LICENSE](LICENSE) and
[THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md) for the licences of the Go
modules linked into the binary.
