# Harness catalog and certification contract

The catalog describes public CLI contracts; inventory records local normalized
observations; an execution template states portable intent; and a binding
records operator-confirmed project routing. These four artifacts are separate
and follow the schemas in `docs/schemas/`.

The curated catalog is assembled deterministically from `catalog/harnesses.json`
and sorted JSON descriptors in `catalog/harnesses.d/`. Descriptor IDs must be
globally unique. Adding a harness changes data and its contract tests, not the
resolver. Deck freshness is exchanged through `docs/deck-sync-contract.md`; an
uncataloged Deck observation is visible but not routable.

Standalone discovery also loads the bounded allowlist in
`catalog/harness-candidates.json`. Each row names exact executable aliases, a
five-second `--version` argv, and a strict version pattern. Installed rows are
visible as unverified inventory candidates but have no models or execution
contract, so they cannot enter routing. Missing executables produce no row;
aliases collapse to one stable harness ID. This list is not PATH scanning and
does not read provider configuration.

## Harness descriptor

A descriptor has a stable harness ID, executable name, exact argv-vector
templates, bounded probes, normalized capabilities, version ranges, and
certification evidence. No field may contain shell text, an executable path,
provider configuration, a credential source, an account identifier, or raw
probe output.

Argument construction substitutes only typed placeholders into array elements:
`{model}`, `{reasoning}`, `{workspace}`, and `{prompt}`. It never joins a shell
command. Unsupported controls are absent, not approximated. A descriptor must
state whether noninteractive mode auto-approves tool use and what external
isolation it requires.

Allowed probes are:

- executable lookup by the operating system;
- an exact version argv with a five-second timeout and a strict version regex;
- an exact, documented, read-only model enumeration argv with a ten-second
  timeout; and
- an optional readiness probe only when the CLI documents a sanitized result
  that does not expose auth material, identity, quota, balance, or config.

Probe stdin is closed; prompts and timeouts fail; stdout and stderr are capped;
only the matched version and strict public model-selector shapes survive
sanitization. Enumerated selectors absent from the curated catalog remain
visible with unknown family, tier, entitlement, and reasoning; they are not
routable until a descriptor classifies them.
The implementation does not read process environments or provider files.
Standalone results are labeled `standalone-best-effort`; executable presence
and sanitized model enumeration do not prove readiness. An explicitly enabled
Deck integration may supply `deck-runtime` inventory for Deck's own adapters.
It remains separate from compatibility certification.

## Normalized capability vocabulary

Harness ranges and models may claim only these initial capabilities:

- `noninteractive`
- `structured-output`
- `reasoning-level`
- `ephemeral-session`
- `resume`
- `read-only-tools`
- `permission-boundary`
- `model-enumeration`

Curated models additionally declare `modelFamily`, `qualityTier` (`economy`,
`balanced`, `frontier`), supported reasoning levels, and an entitlement class
(`included`, `metered`, `unknown`). Cost is an ordering class, not a price or
quota claim. Unknown values remain unknown.

## Inventory states

Each harness observation advances independently through:

1. `absent`: no executable found;
2. `installed`: executable and parsed version found;
3. `matched`: version is inside a descriptor range;
4. `ready`: an approved readiness probe succeeded;
5. `enumerated`: an approved enumeration probe produced sanitized model IDs;
6. `operator-declared`: a public capability was supplied in the answers file;
7. `unverified`: installed version or declared model lacks current contract
   evidence.

States do not imply authentication, spend authority, quota, or runtime health.
For an installed harness without a safe enumeration command, catalog models are
shown with `catalog-static` provenance and unknown entitlement. They remain
non-routable unless the operator declares them. This makes the catalog visible
without representing account availability as observed fact.

Live subscription quota is outside this contract entirely. It is not a catalog
fact, not an inventory field, and not a synchronized one: it belongs to
whatever runtime holds the provider session, it changes between one probe and
the next, and it is account data the exported bundle already forbids. An
entitlement class says whether a plan covers a model at all; it never says how
much of that plan is left.

The disposable inventory follows
[`inventory.schema.json`](schemas/inventory.schema.json), is ignored by default,
and is never copied wholesale into adoption files.

## Execution template and binding

The template follows
[`execution-template.schema.json`](schemas/execution-template.schema.json). A
role contains hard constraints and an ordered candidate list. The resolver
filters hard constraints first and then applies cost/capability preference.
Formal review must be a different model family from every applicable
implementation route. Repair escalates after exactly two failed review rounds
to the next quality tier.

The committed binding contains only catalog version, template ID, public
harness/model IDs, normalized reasoning level, provenance, fallbacks,
constraints, and unresolved roles. It contains no launch token, prompt,
workspace path, inventory snapshot, price, credential, or provider state.

## Certification

A version range is `supported` only when one immutable certification record
contains all applicable evidence. The range is a compatibility envelope, not
an inheritance rule: only an exact detected version listed in
`certification.testedVersions` is supported and routable. Every tested version
must be valid semver within the declared range, and the record binds its UTC
certification time and result digest:

1. schema validation for descriptor and normalized capabilities;
2. exact argv-vector tests for model, reasoning, noninteractive, structured
   output, permissions, workspace, and prompt placement;
3. hermetic fake-executable tests for valid versions, boundary versions,
   malformed output, oversized output, prompts, timeouts, signals, non-zero
   exits, sanitizer rejection, and zero credential/config reads;
4. resolver tests for ranking, independence, fallback, unresolved roles, and
   two-round escalation;
5. explicitly enabled live discovery and no-op model-selection tests against
   every recorded CLI version; and
6. Deck import/launch interoperability evidence when a stable Deck contract is
   accepted.

Ordinary offline tests must set `live: false` and cannot invoke a model, contact
a provider, consume quota, spend money, install software, authenticate, or
read credential/config state. Live tests require separate operator authority,
a named non-secret budget reference, and an expiry; the catalog stores only
the tested public CLI version, public model ID, date, and result digest.

Catalog validation rejects tested versions outside their declared range.
Missing Deck interoperability evidence is `missing` under the version-1 sync
contract. Missing live evidence makes a range `experimental`, never
`supported`. An installed version outside a supported range is `unverified`.
Patch-version drift is not assumed compatible.

## Initial incubation rows

The implementation includes experimental descriptors for `claude`, `cline`,
`codex`, `command-code`, `devin`, `kilo`, `kimi`, `mimo`, `omp`, `opencode`,
`pi`, and `qwen`, based on installed help and official documentation. `cline`,
`mimo`, `pi`, and `qwen` have no approved read-only enumeration probe (`pi`'s
`--list-models` table splits the provider and model into separate columns, so
the sanitizer cannot reconstruct the `provider/id` selector), so their catalog
models remain `catalog-static` and non-routable until the operator declares
them. Pi and OMP shared a pi-tui autoscroll defect that was resolved with
GLM 5.3. The balanced template may route
through Pi only after an operator declaration; OMP can use bounded safe model
enumeration. The discovery candidate
list is currently empty. These rows
enable hermetic development and honest inventory, but they remain experimental
until authorized live certification. Their presence is not a support claim.
